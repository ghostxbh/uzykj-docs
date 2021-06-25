---
title: SpringBoot读取配置值
date: 2019-08-08
tags:
    - SpringBoot
author: ghostxbh
location: blog
summary: 介绍SpringBoot读取配置的几种方式。
---
# 读取配置值

### 方法一：
`@Value`注解的方式取值

+ 设定appliction.properties的配置信息

```yaml
xiaoming.sex=boy
xiaoming.age=18
xiaoming.score=98
```

+ 使用@Value取值

```java
@RestController
public class PersonController {
    @Value("${xiaoming.sex}")
    private String sex;
    @Value("${xiaoming.age}")
    private Integer age;
    @Value("${xiaoming.score}")
    private Integer score;

    @RequestMapping("/xiaoming")
    public String get() {
        return String.format("小明==》性别：%s-----年龄：%s-----分数：%s",sex,age,score);
    }
}
```

+ 页面展示

小明==》性别：boy-----年龄：18-----分数：98

### 方法二：
使用`@ConfigurationProperties`赋值给实体类

+ 设定appliction.yml的配置信息

```yaml
person:
  name: xiaoming
  age: 18
```

+ `@ConfigurationProperties`赋值给实体类

```java
@Component
@ConfigurationProperties(prefix = "person")
public class Person {
    private String name;

    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

}
```

+ 请求信息

```java
@Autowired
private Person person;

@RequestMapping("/person")
public String getPerson() {
    return String.format("姓名：%s-----年龄：%s",person.getName(),person.getAge());
}
```

+ 页面展示

姓名：xiaoming-----年龄：18

### 方法三：
通过注入获取Environment对象，然后再获取定义在配置文件的属性值

+ 设定`appliction.properties`的配置信息

```yml
springboot.test=hello-springboot
```

+ 获取Environment对象，然后再获取定义在配置文件的属性值

```java
private static final String hello = "springboot.test";

@Autowired
private Environment environment;

@RequestMapping("/enviro")
public String getenv() {
    return String.format("测试Environment：" + environment.getProperty(hello));
}
```
 
+ 页面展示

测试Environment：hello-springboot

### 源码地址
[springboot-value](https://github.com/ghostxbh/spring-boot-example/tree/master/boot01)

---
收录时间: 2019-08-08

<Vssue :title="$title" />
