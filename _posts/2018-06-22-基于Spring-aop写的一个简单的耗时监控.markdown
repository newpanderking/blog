---
layout: post
title:  "基于Spring aop写的一个简单的耗时监控"
categories: 技术
tags: spring
author: jason
description: 基于spring aop思想做一个切面监控，监控应用方法耗时情况
---


> 背景: 在做项目的时候，大家肯定都遇到对一些对方法，模块耗时的监控，为了方便性能的监控，问题的定位等。如果每一个方法里都加上

```java
...
Stopwatch watch = Stopwatch.createStarted();
...
watch.elapsed(TimeUnit.MILLISECONDS)
...

```
> 类似的代码肯定没问题，但是就会显得代码特别的冗余。正好近期看了点spring-aop的相关数据，就想到做一个对方法的切面监控方法耗时，同时利用一个简单的注解，可以达到代码简洁化。

* 1、定义注解类

```java
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;

@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface PerfMonitor {

    /**
     * 基本描述，可不设置，默认为空
     * @return
     */
    String desc() default "";
}

```

* 2、定义切面

```java
import java.lang.reflect.Method;
import java.util.concurrent.TimeUnit;

import com.google.common.base.Stopwatch;
import org.aspectj.lang.JoinPoint;
import org.aspectj.lang.Signature;

public class PerfAspect {
    private Logger logger = Logger.getLogger(PerfAspect.class);
    /**
     * 耗时，spring线程问题，防止不同线程之间出现数据影响
     */
    private ThreadLocal<Stopwatch> watch = new ThreadLocal<>();
    /**
     * 进入方法埋点
     */
    public void before() {
        // 设置进入时间
        watch.set(Stopwatch.createStarted());
    }
    /**
     * 结束方法埋点
     * @param point
     */
    public void after(JoinPoint point) {
        Class c = point.getTarget().getClass();
        Signature signature = point.getSignature();
        StringBuilder sb = new StringBuilder();
        String desc = getAnnotationDesc(point);
        if (desc != null && desc != "") {
            sb.append(desc);
        } else {
            sb.append(c.getSimpleName()).append(".").append(signature.getName());
        }   sb.append(",cost[").append(watch.get().elapsed(TimeUnit.MILLISECONDS)).append("]ms");
        logger.info(sb.toString())
    }

    /**
     * 获取注解描述信息
     * @param joinPoint
     * @return
     */
    private String getAnnotationDesc(JoinPoint joinPoint) {
        Method method = getJoinPointMethod(joinPoint);
        PerfMonitor perfMonitor = method.getAnnotation(PerfMonitor.class);
        return perfMonitor.desc();
    }

    /**
     * 计算接入点监控方法
     * @param joinPoint
     * @return
     */
    private Method getJoinPointMethod(JoinPoint joinPoint) {
        Class c = joinPoint.getTarget().getClass();
        Signature signature = joinPoint.getSignature();
        if (c != null && c.getMethods() != null) {
            for (Method method : c.getMethods()) {
                if (method.getName() == signature.getName()) {
                    return method;
                }
            }
        }
        return null;
    }

}

```

* 3、在方法上打上该注解

```java
public class Student implements Person {

    /**
     * logger
     */
    private Logger logger = Logger.getLogger(Student.class);

    @PerfMonitor(desc = "student eat perf")
    public void eat(String food) {
　　
        logger.info("student eat " + food);
    }
}

```

* 4、spring xml配置

```xml
<bean id="student" class="com.common.util.test.Student"/>
<!--性能监控点-->
    <bean id="perfAspect" class="com.common.util.perfmonitor.PerfAspect"/>
    <!-- 开启aop -->
    <aop:aspectj-autoproxy/>
    <!-- aop配置 -->
    <aop:config>
        <!-- 连接点 -->
        <aop:pointcut id="pointCut" expression="@annotation(com.common.util.perfmonitor.PerfMonitor)"/>
        <aop:aspect ref="perfAspect">
            <aop:after method="after" pointcut-ref="pointCut"/>
            <aop:before method="before" pointcut-ref="pointCut"/>
        </aop:aspect>
    </aop:config>
```

* 5、测试入口

```java
public class Client {

  public static void main(String[] args)  {
    ApplicationContext ctx = new ClassPathXmlApplicationContext("spring.aop.xml");
    final Person person = (Person)ctx.getBean("student");
    ExecutorService executorService = Executors.newFixedThreadPool(10);

        for (int i = 0; i < 100; i++) {
            executorService.execute(new Runnable() {
                public void run() {

                    person.eat("fish");
                }
            });

        }
  }
}

```

#### 遇到的问题

* `xml aop` 配置后, 切面不生效，重点关注，aop文件配置加载和要做切面的类加载顺序。
* aop配置扫描范围限于当前bundle，不同bundle需要各自配置。
