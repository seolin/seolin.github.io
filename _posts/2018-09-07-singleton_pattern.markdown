---
layout:     post
title:      "单例模式详解"
subtitle:   " \" 单例模式的几种情况 \""
date:       2018-08-16
author:     "Aziz"
header-img: "img/in-post/singleton_pattern/singleton_pattern.jpg"
tags:
    - design pattern
    - java
---

> “对象世界里面，也可能只有一个太阳的情况”

## 单例的目的
+ 在应用中如果有两个或者两个以上的实例会引起错误（在整个应用中，同一时刻，有且只能有一种状态）
+ 尽可能的节约内存空间，减少无谓的GC消耗，并且使应用可以正常运作

## 单例的实现
1. 静态实例，带有static关键字的属性在每一个类中都是唯一的。
2. 限制客户端随意创造实例，即私有化构造方法，此为保证单例的最重要的一步。
3. 给一个公共的获取实例的静态方法，注意，是静态的方法，因为这个方法是在我们未获取到实例的时候就要提供给客户端调用的，所以如果是非静态的话，那就变成一个矛盾体了，因为非静态的方法必须要拥有实例才可以调用。
4. 判断只有持有的静态实例为null时才调用构造方法创造一个实例，否则就直接返回。

### 最标准以及最简单的单例

```
public class Singleton{
    //一个静态实例
    private static Singleton singleton;
    //私有化构造函数
    private Singleton(){}
    //给出一个公共的静态方法返回一个单一实例
    public static Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}

```
但是上述会有线程安全的问题，在多线程的情况下，可能会有多个实例被创建

### 粗暴的单例

```
public class Singleton{

    //一个静态实例
    private static Singleton singleton;
    //私有化构造函数
    private Singleton(){}
    //给出一个公共的静态方法返回一个单一实例
    public static synchronized Singleton getInstance(){
        if(singleton == null){
            singleton = new Singleton();
        }
        return singleton;
    }
}

```
该方式效率比较低下

### 双重锁定的单例

```
public class Singleton{
    //一个静态实例
    private static Singleton singleton;
    //私有化构造函数
    private Singleton(){}
    //给出一个公共的静态方法返回一个单一实例
    public static Singleton getInstance(){
        if(singleton == null){
            sychronized(Singleton.class){
                if(singleton == null){
                      singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```
该方式在jvm层面，可能会出现问题

+ jvm处理创建新对象方式
1. 分配内存
2. 初始化构造器
3. 将对象指向分配的内存的地址

jvm可能会将2，3步骤返回来执行（2和3可能是相反的是因为JVM会针对字节码进行调优，而其中的一项调优便是调整指令的执行顺序）。
如果出现这种情况，会有莫名其妙的错误，如果加上volatile关键字就能解决这个问题

### 最优的实现方式
```
public class Singleton{
    public static Singleton getInstance(){
        return SingletonInstance.instance;
    }
    private static class SingletonInstance{
        public static Singleton instance;
    }
}

```
一个类的静态属性只会在第一次加载类时初始化，这是JVM帮我们保证的，所以我们无需担心并发访问的问题。所以在初始化进行一半的时候，别的线程是无法使用的，因为JVM会帮我们强行同步这个过程。另外由于静态变量只初始化一次，所以singleton仍然是单例的。
1. Singleton最多只有一个实例，在不考虑反射强行突破访问限制的情况下。
2. 保证了并发访问的情况下，不会发生由于并发而产生多个实例，不会由于初始化动作未完全完成而造成使用了尚未正确初始化的实例。

### 多线程测试
```
public class TestSimpleSinglton {
    boolean lock;

    public boolean isLock() {
        return lock;
    }

    public void setLock(boolean lock) {
        this.lock = lock;
    }

    public static void main(String[] args) throws InterruptedException {
        final Set<String> singletonInstanceSet = Collections.synchronizedSet(new HashSet<String>());
        final TestSimpleSinglton lock = new TestSimpleSinglton();
        lock.setLock(true);
        ExecutorService executorService = Executors.newCachedThreadPool();
        for (int i = 0; i < 1000; i++) {
            final int num = i;
            executorService.execute(new Runnable() {
                @Override
                public void run() {
                    System.out.println("this is the num " + num);
                    while (true) {
                        if (!lock.isLock()) {
                            Singleton singleton = Singleton.getInstance();
                            singletonInstanceSet.add(singleton.toString());

                            break;
                        } else {
                            System.out.println("本次不添加");
                        }
                    }
                }
            });
        }
        Thread.sleep(10000);
        lock.setLock(false);
        System.out.println("------将开关打开-----");
        Thread.sleep(10000);
        System.out.println("-------并发情况下获取到我们的单例实例----------");
        for (String instance : singletonInstanceSet) {
            System.out.println(instance);
        }
        executorService.shutdown();
    }
}
```
## 结论
目前为止，使用单例模式最好就是用静态的最终形式

## 参考
[单例模式详解](https://www.cnblogs.com/zuoxiaolong/p/pattern2.html)