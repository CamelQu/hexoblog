---
title: Java中的不可变对象（immutable object）
date: 2016-04-30 21:40:00
tags:
comments: true
---
# 定义
近期写代码的过程中遇到了可变对象和不可变对象的问题，因此找了一些资料来研究下。首先看下Oracle对于不可变对象（immutable object）的定义与示例。
> 当一个对象构建好后其状态不再会被改变，那么这个对象就可以被称作不可变对象。最大程度地依靠不可变对象是在编写简单可依赖代码的一个被广泛接受的成熟策略。

> 不可变对象在并发应用当中十分有用。鉴于它们的状态不能被改变，不可变对象不会被线程干扰毁坏或者被观察时处于不一致的状态。

> 程序员通常不愿意使用不可变对象，因为他们担心用对象的创建替换对象的更新的成本。对象创建的影响通常被过高的估计，并且可以被使用不可变对象相关的一些效率进行补偿。这包括由于垃圾回收而降低的开销以及去除了为保护可变对象不受污染的代码。

这段<a href="">说明</a>不仅给出了不可变对象的定义，还指出了不可变对象的优点：多线程并发下的同步性。根据Oracle中给出的不可变对象的例子：
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
