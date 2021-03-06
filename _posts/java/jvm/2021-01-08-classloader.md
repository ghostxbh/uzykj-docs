---
title: ClassLoader类加载器
date: 2021-01-08
tags:
    - ClassLoader
author: 编程界的小学生
location: Gitee
summary: 本文介绍ClassLoader类加载器的加载过程、种类、范围，双亲委派等。
---
# ClassLoader 类加载器

## 1、类加载过程

![类加载时机](http://file.uzykj.com/8678c15f-1242-84cf-7e19-321a52b2def7.png)

- 加载

> 将类的.class文件中的二进制数据读入到内存中，将其放在运行时数据区的方法区内，然后在内存上创建一个`java.lang.Class`对象用来封装类在方法区内的数据结构作为这个类的各种数据的访问入口。

- 验证

> 主要是为了确保class文件中的字节流包含的信息是否符合当前JVM的要求，且不会危害JVM自身安全，比如校验文件格式、是否是cafe baby魔数、字节码验证等等。

- 准备

> 为类变量分配内存并设置类变量（是被static修饰的变量，变量不是常量，所以不是final的，就是static的）初始值的阶段。这些变量所使用的内存在方法区中进行分配。比如
>
> ```
> private static int age = 26;
> ```
>
> 类变量age会在准备阶段过后为 其分配四个（int四个字节）字节的空间，并且设置初始值为0，而不是26。
>
> 若是final的，则在编译期就会设置上最终值。

- 解析

> JVM会在此阶段把类的二进制数据中的符号引用替换为直接引用。

- 初始化

> 初始化阶段是执行类构造器`<clinit>()`方法的过程，到了初始化阶段，才真正开始执行类定义的Java程序代码（或者说字节码 ）。比如准备阶段的那个age初始值是0，到这一步就设置为26。

- 使用

> 对象都出来了，业务系统直接调用阶段。

- 卸载

> 用完了，可以被GC回收了。

## 2、类加载器种类以及加载范围

![类加载器种类](http://file.uzykj.com/7f423103-4198-a249-0ac1-f969aff88b0e.png)

- 启动类加载器（Bootstrap ClassLoader）

> 最顶层类加载器，他的父类加载器是个null，也就是没有父类加载器。负责加载jvm的核心类库，比如`java.lang.*`等，从系统属性中的`sun.boot.class.path`所指定的目录中加载类库。他的具体实现由Java虚拟机底层C++代码实现。

- 扩展类加载器（Extension ClassLoader）

> 父类加载器是Bootstrap ClassLoader。从`java.ext.dirs`系统属性所指定的目录中加载类库，或者从JDK的安装目录的`jre/lib/ext`子目录（扩展目录）下加载类库，如果把用户的jar文件放在这个目录下，也会自动由扩展类加载器加载。继承自`java.lang.ClassLoader`。

- 应用程序类加载器（Application ClassLoader）

>  父类加载器是Extension ClassLoader。从环境变量classpath或者系统属性`java.class.path`所指定的目录中加载类。继承自`java.lang.ClassLoader`。

- 自定义类加载器（User ClassLoader）

> 除了上面三个自带的以外，用户还能制定自己的类加载器，但是所有自定义的类加载器都应该继承自`java.lang.ClassLoader`。比如热部署、tomcat都会用到自定义类加载器。

- 补充：不同ClassLoader加载的文件路径配置在如下源码里写的：

```java
// sun.misc.Launcher

public class Launcher {
    // Bootstrap类加载器的加载路径，在static静态代码块里用的
    private static String bootClassPath = System.getProperty("sun.boot.class.path");
    
    // AppClassLoader 继承 ClassLoader
    static class AppClassLoader extends URLClassLoader {
        public static ClassLoader getAppClassLoader(final ClassLoader var0) throws IOException {
            // java.class.path
            final String var1 = System.getProperty("java.class.path");
        }
    }
    
    // ExtClassLoader 继承 ClassLoader
    static class ExtClassLoader extends URLClassLoader {
        public static Launcher.ExtClassLoader getExtClassLoader() throws IOException {
            // java.ext.dirs
            String var0 = System.getProperty("java.ext.dirs");
        }
    }   
}
```

## 3、双亲委派是什么

如果一个类加载器收到了类加载的请求，他首先会从自己缓存里查找是否之前加载过这个class，加载过直接返回，没加载过的话他不会自己亲自去加载，他会把这个请求委派给父类加载器去完成，每一层都是如此，类似递归，一直递归到顶层父类，也就是`Bootstrap ClassLoader`，只要加载完成就会返回结果，如果顶层父类加载器无法加载此class，则会返回去交给子类加载器去尝试加载，若最底层的子类加载器也没找到，则会抛出`ClassNotFoundException`。

源码在`java.lang.ClassLoader#loadClass(java.lang.String, boolean)`

![双亲委派模型](http://file.uzykj.com/5e5253e3-2e6b-58f1-228b-697bb7377dfa.png)

## 4、为啥要有双亲委派

防止内存中出现多份同样的字节码，安全。

比如自己重写个`java.lang.Object`并放到Classpath中，没有双亲委派的话直接自己执行了，那不安全。双亲委派可以保证这个类只能被顶层`Bootstrap Classloader`类加载器加载，从而确保只有JVM中有且仅有一份正常的java核心类。如果有多个的话，那么就乱套了。比如相同的类`instance of`可能返回false，因为可能父类不是同一个类加载器加载的Object。

## 5、为什么需要破坏双亲委派模型

- Jdbc

> Jdbc为什么要破坏双亲委派模型？
>
>以前的用法是未破坏双亲委派模型的，比如`Class.forName("com.mysql.cj.jdbc.Driver");`
>
>而在JDBC4.0以后，开始支持使用spi的方式来注册这个Driver，具体做法就是在mysql的jar包中的`META-INF/services/java.sql.Driver`文件中指明当前使用的Driver是哪个，然后使用的时候就不需要我们手动的去加载驱动了，我们只需要直接获取连接就可以了。`Connection con = DriverManager.getConnection(url, username, password ); `
>
>首先，理解一下为什么JDBC需要破坏双亲委派模式，原因是原生的JDBC中Driver驱动本身只是一个接口，并没有具体的实现，具体的实现是由不同数据库类型去实现的。例如，MySQL的`mysql-connector-*.jar`中的Driver类具体实现的。 原生的JDBC中的类是放在`rt.jar`包的，是由Bootstrap加载器进行类加载的，在JDBC中的Driver类中需要动态去加载不同数据库类型的Driver类，而`mysql-connector-*.jar`中的Driver类是用户自己写的代码，那Bootstrap类加载器肯定是不能进行加载的，既然是自己编写的代码，那就需要由Application类加载器去进行类加载。这个时候就引入线程上下文件类加载器(`Thread Context ClassLoader`)，通过这个东西程序就可以把原本需要由Bootstrap类加载器进行加载的类由Application类加载器去进行加载了。

- Tomcat

>  Tomcat为什么要破坏双亲委派模型？
>
> 因为一个Tomcat可以部署N个web应用，但是每个web应用都有自己的classloader，互不干扰。比如web1里面有`com.test.A.class`，web2里面也有`com.test.A.class`，如果没打破双亲委派模型的话，那么web1加载完后，web2在加载的话会冲突。因为只有一套classloader，却出现了两个重复的类路径，所以tomcat打破了，他是线程级别的，不同web应用是不同的classloader。

- Java spi 方式，比如jdbc4.0开始就是其中之一。

- 热部署的场景会破坏，否则实现不了热部署。

## 6、如何破坏双亲委派模型

重写`loadClass`方法，别重写`findClass`方法，因为`loadClass`是核心入口，将其重写成自定义逻辑即可破坏双亲委派模型。

## 7、如何自定义一个类加载器

只需要继承`java.lang.Classloader`类，然后覆盖他的`findClass(String name)`方法即可，该方法根据参数指定的类名称，返回对应 的Class对象的引用。

## 8、热部署原理

采取破坏双亲委派模型的手段来实现热部署，默认的`loadClass()`方法先找缓存，你改了class字节码也不会热加载，所以自定义ClassLoader，去掉找缓存那部分，直接就去加载，也就是每次都重新加载。

## 9、如何对“.class”文件处理保证不被人拿到以后反编译获取公司源代码？
首先你编译时，就可以采用一些小工具对字节码加密，或者做混淆等处理。
现在有很多第三方公司，都是专门做商业级的字节码文件加密的，所以可以付费购买他们的产品。
然后在类加载的时候，对加密的类，考虑采用自定义的类加载器来解密文件即可，这样就可以保证你的源代码不被人窃取，网上也有很多成熟的开源插件进行加解密。

## 10、包含main方法的类会优先加载，如果一个项目中有多个类中都有main方法，都会加载么？

不会的，你启动一个jar包，需要指定某个main主类，优先就是加载他，其他类里的main方法不会被加载，所以没有规定说不建议写多个main。

## 11、类加载器是把jar包里的所有类一次性全部加载进去吗？

不是的，首先加载包含main方法的主类，接着是运行你写的代码的时候，遇到你用了什么类，再加载什么类。

## 12、常见笔试题

问题：输出结果是什么？

答案：编译报错。

原因：因为静态语句块中只能访问定义在静态语句块之前的变量，定义在他之后的 变量在前面的静态语句块中可以赋值，但是不能访问。

```java
/**
 * Description: 编译报错
 *
 * @author TongWei.Chen 2021-01-08 17:37:44
 */
public class Test1 {
    static {
        // 编译没报错
        i = 2;
        // 编译报错Illegal forward reference
        System.out.println(i);
    }
    private static int i =1;
}
```

问题：输出结果是什么？

答案 ：1、3

原因：因为类加载过程中会先准备类变量（也就是静态变量），准备阶段是赋初始值阶段，也就是`test2=null，value1=0，value2=0`，然后进入初始化阶段的时候`test2=new Test2()`，会执行构造器，结果是`value1 = 1，value2 = 4`，然后执行value1和value2这两句，value1没变化，value2被重新赋值成了3，所以结果1和3。

```java
public class Test2 {
    private static Test2 test2 = new Test2();
    private static int value1;
    private static int value2 = 3;

    private Test2() {
        value1 ++;
        value2 ++;
    }

    public static void main(String[] args) {
        // 1
        System.out.println(test2.value1);
        // 3
        System.out.println(test2.value2);
    }
}
```

那如果把`private static Test2 test2 = new Test2();`放到`private static int value2 = 3;`下面的话结果就是1和4了。

```java
public class Test3 {
    private static int value1;
    private static int value2 = 3;
    private static Test3 test3 = new Test3();
    
    private Test3() {
        value1 ++;
        value2 ++;
    }
    
    public static void main(String[] args) {
        // 1
        System.out.println(test3.value1);
        // 4
        System.out.println(test3.value2);
    }
}
```

---
收录时间: 2021-01-08

<Vssue :title="$title" />