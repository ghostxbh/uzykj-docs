# 并发特性：可见性

### 定义：
当一个线程修改了共享变量的值，其他线程能够看到修改的值。

### 场景：
+ 存在2个线程，一个共享变量`count`
+ 2个线程都去调用`count`累加，并小于10000的值

### 代码示例：

+ 共享变量
```
private static long count = 0;
```

+ 2个线程调用`count`累加
```
public static long acdc() throws InterruptedException {
    final VisibleTest poolTest = new VisibleTest();
    Thread thread1 = new Thread(() -> poolTest.add10K());
    Thread thread2 = new Thread(() -> poolTest.add10K());

    thread1.start();
    thread2.start();

    thread1.join();
    thread2.join();
    return count;
}
```

### 输出
```
输出： 10000 < count < 20000
```

### 技术保障

在Java中提供了多种可见性保障措施，这里主要涉及四种：
+ 通过`volatile`关键字标记内存屏障保证可见性。
+ 通过`synchronized`关键字定义同步代码块或者同步方法保障可见性。
+ 通过`Lock`接口保障可见性。
+ 通过`Atomic`类型保障可见性。

#### volatile
`volatile` 关键字并不是`java`语言的特产，古老的C语言也有，最原始的意义就是禁用CPU缓存。

被`volatile`关键字修饰的变量，在每个写操作之后，都会加入一条`store`内存屏障命令，此命令强制工作内存将此变量的最新值保存至主内存；
在每个读操作之前，都会加入一条`load`内存屏障命令，此命令强制工作内存从主内存中加载此变量的最新值至工作内存。
```java
public class VisibleVolatile {
    private volatile static boolean tag;

    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            System.out.println("thread1 is running");
            while (!tag) ;
            System.out.println("thread1 is end");
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("thread2 is running");
            tag = true;
            System.out.println("thread2 is end");
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
    }
}
```

```
// 输出
thread1 is running
thread2 is running
thread1 is end
thread2 is end
```


---
收录时间: 2018/09/12

<Vssue :title="$title" />
