---
title: 基于接口SchedulingConfigurer的动态定时任务
toc: true
date: 2022-03-25 10:33:01
tags: spring
categories: spring
---
一般配置都是基于注解(@Scheduled)的简单定时器，其使用固定的cron表达式，现在想使用动态方式配置任务周期。

## 1. 固定的简单定时器
如下配置：
```

package com.example.schedule.SimpleSchedule;

import org.springframework.scheduling.annotation.Scheduled;

import java.time.LocalDateTime;

/**
 * @fileName：Schedule
 * @createTime：2019/5/14 17:16
 * @author：
 * @version：
 * @description：基于注解(@Scheduled)的简单定时器demo
 *
 * cron表达式语法:[秒] [分] [小时] [日] [月] [周] [年]
 * @Scheduled(fixedDelay = 5000) //上一次执行完毕时间点之后5秒再执行
 * @Scheduled(fixedDelayString = "5000") //上一次执行完毕时间点之后5秒再执行
 * @Scheduled(fixedRate = 5000) //上一次开始执行时间点之后5秒再执行
 * @Scheduled(initialDelay=1000, fixedRate=5000) //第一次延迟1秒后执行，之后按fixedRate的规则每5秒执行一次
 *
 */

//1.主要用于标记配置类
@Configuration
// 2.开启定时任务
@EnableScheduling
public class Schedule {
    //3.添加定时任务
    @Scheduled(cron = "0/5 * * * * ?")
    //或直接指定时间间隔，例如：5秒
    //@Scheduled(fixedRate=5000)

    private void configureTasks() {
        System.err.println("基于注解(@Scheduled)的简单定时器demo: " + LocalDateTime.now());
    }
}

```

## 2. 基于接口SchedulingConfigurer的动态定时任务
此种方法实现SchedulingConfigurer 类，采用多线程方式跑定时任务
```

/**
 * @author ljj
 */
@Component
@EnableScheduling
@Slf4j
public class ResourceBackupTask implements SchedulingConfigurer {

    @Autowired
    private ProjectService projectService;

    @Value("${egova.wukong.project.backup.enabled:true}")
    private boolean projectBackupEnable;

    @Value("${egova.wukong.project.backup.cron:0 0 2 * * ? }")
    private String projectBackupCron;

    @Override
    public void configureTasks(ScheduledTaskRegistrar scheduledTaskRegistrar) {
        //增加任务 addTriggerTask可继续添加任务
        scheduledTaskRegistrar.addTriggerTask(this::projectBackupTask, triggerContext -> {
            CronTrigger cronTrigger = new CronTrigger(projectBackupCron);
            return cronTrigger.nextExecutionTime(triggerContext);
        });
    }

    /**
     * 项目备份逻辑
     */
    void projectBackupTask() {
        if (!projectBackupEnable) {
            return;
        }
        log.info("开始项目备份任务, cron配置为：{}", projectBackupCron);
        int size = projectService.projectBackupTask();
        log.info("项目备份任务结束，完成 {}个项目的备份", size);
    }

}
```