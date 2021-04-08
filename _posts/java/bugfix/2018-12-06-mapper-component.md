---
title: Mapper Component扫描异常
date: 2018-12-06
tags:
    - SpringBoot
author: ghostxbh
location: blog
summary: Mapper Component扫描异常，及解决方案
---
# Mapper Component扫描异常

### 异常信息
```
org.springframework.beans.factory.UnsatisfiedDependencyException: Error creating bean with name 'club.aiit.springboot.Springboot05ApplicationTests': Unsatisfied dependency expressed through field 'userMapper'; nested exception is org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'club.aiit.dao.UserMapper' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}

	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:596)
	at org.springframework.beans.factory.annotation.InjectionMetadata.inject(InjectionMetadata.java:90)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor.postProcessProperties(AutowiredAnnotationBeanPostProcessor.java:374)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.populateBean(AbstractAutowireCapableBeanFactory.java:1378)
	at org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory.autowireBeanProperties(AbstractAutowireCapableBeanFactory.java:396)
	at org.springframework.test.context.support.DependencyInjectionTestExecutionListener.injectDependencies(DependencyInjectionTestExecutionListener.java:119)
	at org.springframework.test.context.support.DependencyInjectionTestExecutionListener.prepareTestInstance(DependencyInjectionTestExecutionListener.java:83)
	at org.springframework.boot.test.autoconfigure.SpringBootDependencyInjectionTestExecutionListener.prepareTestInstance(SpringBootDependencyInjectionTestExecutionListener.java:44)
	at org.springframework.test.context.TestContextManager.prepareTestInstance(TestContextManager.java:246)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.createTest(SpringJUnit4ClassRunner.java:227)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner$1.runReflectiveCall(SpringJUnit4ClassRunner.java:289)
	at org.junit.internal.runners.model.ReflectiveCallable.run(ReflectiveCallable.java:12)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.methodBlock(SpringJUnit4ClassRunner.java:291)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:246)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.runChild(SpringJUnit4ClassRunner.java:97)
	at org.junit.runners.ParentRunner$3.run(ParentRunner.java:290)
	at org.junit.runners.ParentRunner$1.schedule(ParentRunner.java:71)
	at org.junit.runners.ParentRunner.runChildren(ParentRunner.java:288)
	at org.junit.runners.ParentRunner.access$000(ParentRunner.java:58)
	at org.junit.runners.ParentRunner$2.evaluate(ParentRunner.java:268)
	at org.springframework.test.context.junit4.statements.RunBeforeTestClassCallbacks.evaluate(RunBeforeTestClassCallbacks.java:61)
	at org.springframework.test.context.junit4.statements.RunAfterTestClassCallbacks.evaluate(RunAfterTestClassCallbacks.java:70)
	at org.junit.runners.ParentRunner.run(ParentRunner.java:363)
	at org.springframework.test.context.junit4.SpringJUnit4ClassRunner.run(SpringJUnit4ClassRunner.java:190)
	at org.junit.runner.JUnitCore.run(JUnitCore.java:137)
	at com.intellij.junit4.JUnit4IdeaTestRunner.startRunnerWithArgs(JUnit4IdeaTestRunner.java:68)
	at com.intellij.rt.execution.junit.IdeaTestRunner$Repeater.startRunnerWithArgs(IdeaTestRunner.java:47)
	at com.intellij.rt.execution.junit.JUnitStarter.prepareStreamsAndStart(JUnitStarter.java:242)
	at com.intellij.rt.execution.junit.JUnitStarter.main(JUnitStarter.java:70)
Caused by: org.springframework.beans.factory.NoSuchBeanDefinitionException: No qualifying bean of type 'club.aiit.dao.UserMapper' available: expected at least 1 bean which qualifies as autowire candidate. Dependency annotations: {@org.springframework.beans.factory.annotation.Autowired(required=true)}
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.raiseNoMatchingBeanFound(DefaultListableBeanFactory.java:1644)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.doResolveDependency(DefaultListableBeanFactory.java:1203)
	at org.springframework.beans.factory.support.DefaultListableBeanFactory.resolveDependency(DefaultListableBeanFactory.java:1164)
	at org.springframework.beans.factory.annotation.AutowiredAnnotationBeanPostProcessor$AutowiredFieldElement.inject(AutowiredAnnotationBeanPostProcessor.java:593)
	... 28 more
```

### 问题分析
结构问题：

application运行时，需扫描同级子包，如果mapper没有跟dao在一个父级目录下，会找不到这个mapper；

### 解决方案

在开发中我们知道Spring Boot默认会扫描启动类同包以及子包下的注解，那么如何进行改变这种扫描包的方式呢，原理很简单就是：
`@ComponentScan`注解进行指定要扫描的包以及要扫描的类。

接下来我们简单写个例子进行测试下。

第一步：新建两个新包
      我们在项目中新建两个包`cn.kfit`;`org.kfit`；

第二步：新建两个测试类；
在这里为了方便测试，我们让我们的类在启动的时候就进行执行，所以我们就编写两个类，实现接口CommandLineRunner，这样在启动的时候我们就可以看到打印信息了。
`cn.kfit.MyCommandLineRunner1`  : 

```java
package cn.kfit;
 
import org.springframework.boot.CommandLineRunner;
 
@Configuration
publicclass MyCommandLineRunner1 implements CommandLineRunner {
 
    @Override
    publicvoid run(String... args) throws Exception {
       System.out.println("MyCommandLineRunner1.run()");
 
    }
}
org.kfit.MyCommandLineRunner2  : 

package org.kfit;
 
import org.springframework.boot.CommandLineRunner;
 
 
@Configuration
publicclass MyCommandLineRunner2 implements CommandLineRunner {
 
    @Override
    publicvoid run(String... args) throws Exception {
 
       System.out.println("MyCommandLineRunner2.run()");
 
    }
 
}
```


第三步：启动类进行注解指定
在App.java类中加入如下注解：
//可以使用：`basePackageClasses={},basePackages={}`
`@ComponentScan(basePackages={"cn.kfit","org.kfit"})`

 
启动如果看到打印信息：
```
MyCommandLineRunner1.run()
MyCommandLineRunner2.run()
```


说明我们配置成功了。

这时候你会发现，在App.java同包下的都没有被扫描了，所以如果也希望App.java包下的也同时被扫描的话，那么在进行指定包扫描的时候一定要进行指定配置：

`@ComponentScan(basePackages={"cn.kfit","org.kfit","com.kfit"})`


### 资源
[csdn](http://blog.csdn.net/gefangshuai/article/details/50328451)

[iteye](http://412887952-qq-com.iteye.com/blog/2292733)


---
收录时间: 2018-12-06

<Vssue :title="$title" />
