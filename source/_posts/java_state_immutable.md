---
title: Java对象的状态与可变性
date: 2016-06-07 22:45:00
tags: Java
comments: false
---

## 问题引入
近期再看Java的多线程处理，其中有两对概念需要进行一下总结：有状态和无状态，可变类和不可变类。

首先要明确一点，这两对概念都是针对类实例化都的对象而言，因为对象是动态的，而类可以认为是一种数据模板。

## 有状态对象与无状态对象
有状态是指bean可以保存数据，是非线程安全的。而无状态是指bean不能保存数据，是线程安全的。

在这个概念中，数据既可以指实例变量，又可以指<font color="red">其他类的对象引用</font>。

实际上在工作中对对象的处理基本上都是使用spring框架，在spring中，类实例化的默认方式都是单例（singleton），这种方式生成的单例对象都是无状态的，这里由spring处理多线程问题（例如使用TreadLocal）。但是要注意：<font color="red">这并不是说单例就是无状态的，单例是可以为有状态的</font>。同理，scope为原型（Prototype）的类实例化后就是有状态的。

在日常工作中，经常会按照MVC划分代码层次。一般而言，Service与Dao层的类都是无状态的，即仅使用一次。

## 可变对象与不可变对象
首先看下Oracle对于不可变对象（immutable object）的定义与示例。
> 当一个对象构建好后其状态不再会被改变，那么这个对象就可以被称作不可变对象。最大程度地依靠不可变对象是在编写简单可依赖代码的一个被广泛接受的成熟策略。

> 不可变对象在并发应用当中十分有用。鉴于它们的状态不能被改变，不可变对象不会被线程干扰毁坏或者被观察时处于不一致的状态。

> 程序员通常不愿意使用不可变对象，因为他们担心用对象的创建替换对象的更新的成本。对象创建的影响通常被过高的估计，并且可以被使用不可变对象相关的一些效率进行补偿。这包括由于垃圾回收而降低的开销以及去除了为保护可变对象不受污染的代码。

这段<a href="https://docs.oracle.com/javase/tutorial/essential/concurrency/immutable.html">说明</a>不仅给出了不可变对象的定义，还指出了不可变对象的优点：多线程并发下的同步性。根据Oracle中给出的不可变对象的例子：
```
public class SynchronizedRGB {

    // Values must be between 0 and 255.
    private int red;
    private int green;
    private int blue;
    private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public SynchronizedRGB(int red,
                           int green,
                           int blue,
                           String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }

    public void set(int red,
                    int green,
                    int blue,
                    String name) {
        check(red, green, blue);
        synchronized (this) {
            this.red = red;
            this.green = green;
            this.blue = blue;
            this.name = name;
        }
    }

    public synchronized int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public synchronized String getName() {
        return name;
    }

    public synchronized void invert() {
        red = 255 - red;
        green = 255 - green;
        blue = 255 - blue;
        name = "Inverse of " + name;
    }
}
```

在上述的SynchronizedRGB中，存在多线程的并发问题。例如在执行<code>getRGB()</code>与<code>getName()</code>之中对这个类的对象进行改变后，就有可能得到不同的结果。在Oracle官方教程中给出的<a href="https://docs.oracle.com/javase/tutorial/essential/concurrency/syncrgb.html">解决办法</a>是：

```
synchronized (color) {
    int myColorInt = color.getRGB();
    String myColorName = color.getName();
}
```

而使用不可变对象处理上述场景时，可以编写如下的类：
```
final public class ImmutableRGB {

    // Values must be between 0 and 255.
    final private int red;
    final private int green;
    final private int blue;
    final private String name;

    private void check(int red,
                       int green,
                       int blue) {
        if (red < 0 || red > 255
            || green < 0 || green > 255
            || blue < 0 || blue > 255) {
            throw new IllegalArgumentException();
        }
    }

    public ImmutableRGB(int red,
                        int green,
                        int blue,
                        String name) {
        check(red, green, blue);
        this.red = red;
        this.green = green;
        this.blue = blue;
        this.name = name;
    }


    public int getRGB() {
        return ((red << 16) | (green << 8) | blue);
    }

    public String getName() {
        return name;
    }

    public ImmutableRGB invert() {
        return new ImmutableRGB(255 - red,
                       255 - green,
                       255 - blue,
                       "Inverse of " + name);
    }
}
```

在```ImmutableRGB```中，很明显可以看出，所有的成员变量都定义为final类型，并且去除了改变成员变量值的方法的synchronized属性。

尽管Oracle推荐使用不可变对象存储数据，但是显而易见的是，这会产生大量的无用对象，从而加重垃圾回收的负担。
