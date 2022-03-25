---
title: 使用Java8中的parallelStream的一次问题记录
date: 2022-03-22 00:27:51
tags:
toc: true
---
在 java8 中 添加了流Stream，可以让你以一种声明的方式处理数据。使用起来非常简单优雅。ParallelStream 则是一个并行执行的流，采用 ForkJoinPool 并行执行任务，提高执行速度。

<!--more-->

## 1. 现象
压测出现报错
```
2022-03-19 16:32:08
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.211-b12 mixed mode):

"Attach Listener" #147 daemon prio=9 os_prio=0 tid=0x00007fd14c001000 nid=0x708f runnable [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"ForkJoinPool.commonPool-worker-1" #146 daemon prio=5 os_prio=0 tid=0x00007fd0c0057800 nid=0x6370 waiting for monitor entry [0x00007fd0725ea000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at java.util.stream.FindOps$FindTask.makeChild(FindOps.java:268)
	at java.util.stream.FindOps$FindTask.makeChild(FindOps.java:249)
	at java.util.stream.AbstractShortCircuitTask.compute(AbstractShortCircuitTask.java:120)
	at java.util.concurrent.CountedCompleter.exec(CountedCompleter.java:731)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool.helpComplete(ForkJoinPool.java:1870)
	at java.util.concurrent.ForkJoinPool.awaitJoin(ForkJoinPool.java:2045)
	at java.util.concurrent.ForkJoinTask.doInvoke(ForkJoinTask.java:404)
	at java.util.concurrent.ForkJoinTask.invoke(ForkJoinTask.java:734)
	at java.util.stream.FindOps$FindOp.evaluateParallel(FindOps.java:159)
	at java.util.stream.AbstractPipeline.evaluate(AbstractPipeline.java:233)
	at java.util.stream.ReferencePipeline.findAny(ReferencePipeline.java:469)
	at com.egova.wukong.service.impl.PageServiceImpl.lambda$getPageCards$22(PageServiceImpl.java:571)
	at com.egova.wukong.service.impl.PageServiceImpl$$Lambda$1107/1403252310.apply(Unknown Source)
	at java.util.stream.ReferencePipeline$3$1.accept(ReferencePipeline.java:193)
	at java.util.Spliterators$ArraySpliterator.forEachRemaining(Spliterators.java:948)
	at java.util.stream.AbstractPipeline.copyInto(AbstractPipeline.java:481)
	at java.util.stream.AbstractPipeline.wrapAndCopyInto(AbstractPipeline.java:471)
	at java.util.stream.ReduceOps$ReduceTask.doLeaf(ReduceOps.java:747)
	at java.util.stream.ReduceOps$ReduceTask.doLeaf(ReduceOps.java:721)
	at java.util.stream.AbstractTask.compute(AbstractTask.java:316)
	at java.util.concurrent.CountedCompleter.exec(CountedCompleter.java:731)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool$WorkQueue.execLocalTasks(ForkJoinPool.java:1040)
	at java.util.concurrent.ForkJoinPool$WorkQueue.runTask(ForkJoinPool.java:1058)
	at java.util.concurrent.ForkJoinPool.runWorker(ForkJoinPool.java:1692)
	at java.util.concurrent.ForkJoinWorkerThread.run(ForkJoinWorkerThread.java:157)

"ForkJoinPool.commonPool-worker-7" #145 daemon prio=5 os_prio=0 tid=0x00007fd0880ba000 nid=0x632f waiting for monitor entry [0x00007fd0720e5000]
   java.lang.Thread.State: BLOCKED (on object monitor)
	at java.util.stream.FindOps$FindTask.makeChild(FindOps.java:268)
	at java.util.stream.FindOps$FindTask.makeChild(FindOps.java:249)
	at java.util.stream.AbstractShortCircuitTask.compute(AbstractShortCircuitTask.java:119)
	at java.util.concurrent.CountedCompleter.exec(CountedCompleter.java:731)
	at java.util.concurrent.ForkJoinTask.doExec(ForkJoinTask.java:289)
	at java.util.concurrent.ForkJoinPool.helpComplete(ForkJoinPool.java:1870)
	at java.util.concurrent.ForkJoinPool.awaitJoin(ForkJoinPool.java:2045)
	at java.util.concurrent.ForkJoinTask.doInvoke(ForkJoinTask.java:404)
	at java.util.concurrent.ForkJoinTask.invoke(ForkJoinTask.java:734)
```
错误日志指向ForkJoinTask，线程发生阻塞。项目代码
```
 .....

List<CardData> finalCardDataList = cardDataList;
return pageCards.parallelStream()  //第一次并行流
			.sorted(Comparator.comparingInt(PageCard::getLevel).reversed())
			.map(pageCard -> {
				PageCardModel pageCardModel = new PageCardModel();
				pageCardModel.setBase(cards.parallelStream() //第二次并行流
						.filter(card -> StringUtils.equals(card.getId(), pageCard.getCardId())).findAny().orElse(null));
				pageCardModel.setData(finalCardDataList.parallelStream() //第三次并行流
						.filter(cardData -> StringUtils.equals(cardData.getId(), pageCard.getDataId())).findAny().orElse(null));
				pageCardModel.setStyle(pageCard.getStyle());
				pageCardModel.setId(pageCard.getId());
				pageCardModel.setName(pageCard.getName());
				return pageCardModel;
			}).collect(Collectors.toList());
}

parallelStream流内，嵌套两层，并返回大json数据，导致线程占满，IO阻塞
```

## 2. 初步推测
### 并行流陷阱
- 线程安全
	- 由于并行流使用多线程，则一切线程安全问题都应该是需要考虑的问题，如：资源竞争、死锁、事务、可见性等等
- 线程消费
	- 在虚拟机启动时，我们指定了worker线程的数量，整个程序的生命周期都将使用这些工作线程；这必然存在任务生产和消费的问题，如果某个生产者生产了许多重量级的任务（耗时很长），那么其他任务毫无疑问将会没有工作线程可用；更可怕的事情是这些工作线程正在进行IO阻塞。

## 3.设置并行参数
我们可以通过虚拟机启动参数，控制ForkJoinPool的线程数
-Djava.util.concurrent.ForkJoinPool.common.parallelism=N

来设置worker的数量。

## 4. 扩展
parallelStreams()，使用ForkJoinPool。
资源耗尽时使用线程池的默认拒绝策略，在默认的ThreadPoolExecutor.AbortPolicy中，处理程序抛出一个拒绝后运行时RejectedExecutionException。


## 4. 小结
串行流：适合存在线程安全问题、阻塞任务、重量级任务，以及需要使用同一事务的逻辑。

并行流：适合没有线程安全问题、较单纯的数据处理任务。