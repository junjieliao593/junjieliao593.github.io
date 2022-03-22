---
title: 使用Java8中的parallelStream的问题
date: 2022-03-23 00:27:51
tags:
toc: true
---
在 java8 中 添加了流Stream，可以让你以一种声明的方式处理数据。使用起来非常简单优雅。ParallelStream 则是一个并行执行的流，采用 ForkJoinPool 并行执行任务，提高执行速度。

## 现象
出现报错
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
