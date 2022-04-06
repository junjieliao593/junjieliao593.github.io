---
title: springboot异步调用@Async注解
toc: true
date: 2022-04-27 00:13:43
tags:
---


## 背景
当访问其他人的接口较慢或者做耗时任务时，不想程序一直卡在耗时任务上，想程序能够并行执行，我们可以使用多线程来并行的处理任务，也可以使用spring提供的异步处理方式@Async。
## 配置
启动类增加EnableAsync注解
```
...
@EnableAsync
...
public class SingleApplication {
    public static void main(String[] args) {
            SpringApplication.run(SingleApplication.class, args);
    
    }
...
```
## 错误示范
同类方法中使用异步注解没有作用,这样执行后就,是串行单线程
```

package com.web.rest.controller;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
/**
 * 描述: 测试异步调用;
 * 测试多线程调用
 *
 */
@RestController
@RequestMapping("/")
public class AsyncController {
 
    private  final Logger log =  LoggerFactory.getLogger(this.getClass());
 
    @Autowired
    private AsyncControllerTest asyncControllerTest;
 
    /**
     * @Description //方法调用接口类
     * @param
     * @return java.lang.String
     **/
    @GetMapping("test")
    public String test(){
//        asyncControllerTest.doAsyncMethod();
//        asyncControllerTest.doAsyncMethod();
        doAsyncMethod();
        doAsyncMethod();
        log.debug("主线程执行结束......");
        return "SUCCESS";
    }
 
    /**
     * @Description //同类方法中的异步标签
     * @param
     * @return void
     **/
    @Async
    private void doAsyncMethod() {
        try {
            //todo
            Thread.sleep(2000);
            log.debug("异步方法执行了......");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
 
}
```
## 正确做法
改正使用调用另外的类,然后通过注入的方式注入对象:
```

package com.web.rest.controller;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
/**
 * 描述: 测试异步调用;
 * 测试多线程调用
 *
 */
@RestController
@RequestMapping("/")
public class AsyncController {
 
    private  final Logger log =  LoggerFactory.getLogger(this.getClass());
 
    @Autowired
    private AsyncControllerTest asyncControllerTest;
 
    /**
     * @Description //方法调用接口类
     * @param
     * @return java.lang.String
     **/
    @GetMapping("test")
    public String test(){
        asyncControllerTest.doAsyncMethod();
        asyncControllerTest.doAsyncMethod();
//        doAsyncMethod();
//        doAsyncMethod();
        log.debug("主线程执行结束......");
        return "SUCCESS";
    }
 
    /**
     * @Author Young
     * @Description //同类方法中的异步标签
     * @Date 16:45 2019/3/11
     * @param
     * @return void
     **/
    @Async
    private void doAsyncMethod() {
        try {
            //todo
            Thread.sleep(2000);
            log.debug("异步方法执行了......");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
 
}

```

```
package com.web.rest.controller;
 
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.scheduling.annotation.Async;
import org.springframework.stereotype.Component;
 
/**
 * 描述:异步类
 *
 */
@Component
public class AsyncControllerTest {
 
 
    private  final Logger log =  LoggerFactory.getLogger(this.getClass());
 
    @Async
    public void doAsyncMethod() {
        try {
            //todo
            Thread.sleep(2000);
            log.debug("异步方法执行了......");
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```
#### 究其原因:
简单说就是没有过去到代理类,本类调用时,直接自己内部调用,没有走代理类

##### 一般导致@Async失效原因
1.没有在@SpringBootApplication启动类当中添加注解@EnableAsync注解。
2.异步方法使用注解@Async的返回值只能为void或者Future。
3.没有走Spring的代理类。因为@Transactional和@Async注解的实现都是基于Spring的AOP，而AOP的实现是基于动态代理模式实现的。那么注解失效的原因就很明显了，有可能因为调用方法的是对象本身而不是代理对象，因为没有经过Spring容器。

## 升华
#### 如何在本类中调用
这里具体说一下 (导致@Async失效原因) 第三种情况的解决方法。
1.注解的方法必须是public方法。
2.方法一定要从另一个类中调用，也就是从类的外部调用，类的内部调用是无效的。
3.如果需要从类的内部调用，需要先获取其代理类，下面上代码
```

@Service
public class XxxService{
  public void methodA(){
    ...
    XxxService xxxServiceProxy = SpringUtil.getBean(XxxService.class);
    xxxServiceProxy.methodB();
    ...
  }
 
  @Async
  public void methodB() {
    ...
  }
}
```
完整示例：
```

package com.web.rest.controller;
 
import org.aspectj.lang.annotation.Aspect;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.scheduling.annotation.Async;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;
 
import java.util.List;
 
/**
 * @描 述: Manager类Controller接口类
 */
@RestController
@RequestMapping("manager")
public class ManagerController {
    private final Logger logger = LoggerFactory.getLogger(ManagerController.class);
    @Autowired
    private ManagerRepository managerRepository;
//    @Autowired
//    private AsyncController asyncController;
 
    /**
     * @描述 查询所有的数据的异常测试类
     * @参数  []
     **/
    @GetMapping("all")
    public Result getAll(){
        ManagerController bean = SpringUtils.getBean(ManagerController.class);
        logger.debug("查询所有的manager");
        List<ManagerEntity> all = managerRepository.findAll();
        bean.test();
        bean.test();
        return Result.error("数据请求服务未启用","type","subtype");
    }
 
 
    @Async
    public void test(){
        try {
            logger.debug("=======开始执行异步处理类========");
            Thread.sleep(2000);
        }catch (Exception e){
            e.printStackTrace();
        }
    }
 
}
```
SpringUtils的工具类，手动获取bean方法：
```
package com.web.rest.util;
 
import org.springframework.beans.BeansException;
import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationContextAware;
import org.springframework.stereotype.Component;
 
/**
 * @描 述: 手动获取spring的bean
 */
@Component("springContextUtil")
public class SpringUtils implements ApplicationContextAware {
 
 
    private static ApplicationContext applicationContext = null;
 
    public static ApplicationContext getApplicationContext() {
        return applicationContext;
    }
 
    @SuppressWarnings("unchecked")
    public static <T> T getBean(String beanId) {
        return (T) applicationContext.getBean(beanId);
    }
 
    public static <T> T getBean(Class<T> requiredType) {
        return (T) applicationContext.getBean(requiredType);
    }
    /**
     * Spring容器启动后，会把 applicationContext 给自动注入进来，然后我们把 applicationContext
     *  赋值到静态变量中，方便后续拿到容器对象
     * @see org.springframework.context.ApplicationContextAware#setApplicationContext(org.springframework.context.ApplicationContext)
     */
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        SpringUtils.applicationContext = applicationContext;
    }
}
```

#### 补充
Transactional失效情况
@Transactional 加于private方法, 无效
@Transactional 加于未加入接口的public方法, 再通过普通接口方法调用, 无效
@Transactional 加于接口方法, 无论下面调用的是private或public方法, 都有效
@Transactional 加于接口方法后, 被本类普通接口方法直接调用, 无效
@Transactional 加于接口方法后, 被本类普通接口方法通过接口调用, 有效
@Transactional 加于接口方法后, 被它类的接口方法调用, 有效
@Transactional 加于接口方法后, 被它类的私有方法调用后, 有效