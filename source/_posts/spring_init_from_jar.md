---
title: 记一次问题：spring从jar中初始化
date: 2016-06-19 22:37:00
tags: Other
comments: false
---
## 问题引入
<p>由于工程中需要写一个中间件组件公共库，大概就是有一堆类似的中间件放在一起，可以从一个工厂里拿出来，而项目里只需要配置下要用的中间件的名字就可以了，其他的诸如中间件的配置项都放在项目的.properties文件里就好。那么问题来了，由于中间件的初始化都是用spring来完成，那么势必这部分的bean注入配置要放在公共库中，<font color="red">如何让项目读取打包好的公共库中的配置文件呢？</font>

## 分析问题
<p>要解决这个问题，至少要熟悉如下的问题：
>Java是如何加载类的；spring初始化bean的方式是怎样的。

<p>对于第一个问题，实际上已经有很多文章做了解答，这里摘录一些做下记录。<br />
<a href="https://www.ibm.com/developerworks/cn/java/j-lo-classloader/">IBM：深入探讨Java类加载器</a> <br />
<a href="https://zh.wikipedia.org/wiki/Java%E7%B1%BB%E5%8A%A0%E8%BD%BD%E5%99%A8">维基百科：Java类加载器</a>
<p>那么现在需要了解下spring的bean加载机制了。
<p>从使用上说，spring用两种方式来初始化类：xml配置和注解。实际上两种方式大部分时候相同，只不过注解可以简化xml的复杂度。但是xml还是提供来很多灵活的配置方式。例如bean中的init-method，factory-method等，还有就是spEL语句，这些都很实用。spring的类加载在<a href="http://www.cnblogs.com/bobzeng/articles/1877140.html">这片文章</a>中有较为详细的代码分析。
<p>那么在实际的工程项目中（主要是web项目），spring不会主动加载jar中的配置文件，当然需要你主动设置下。那么在上面这个工程中我遇到的问题就是：我在公共库中也Autowired一些对象，但是我的项目不加载库的xml配置文件，所以总提示我NoSuchBeanDefinitionException，即spring在上下文中找不到要创建的这个bean（实际上这个bean的声明在公共库deploy出的第三方jar中）。

## 问题解决
<p>需要明确导入jar中的xml文件。
```
<import resource="classpath*:*.xml" />
```

<p>要注意，这里是使用<font color="red">classpath*</font>，而不是classpath，带上*后spring才会检索第三方jar中指定位置的配置文件。

<p>事实上这次开发还遇到了很多其他的错误，例如：IllegalStateException，NotWritablePropertyException这样的错误，但是当我打开自动注解配置
```
<context:annotation-config />
<context:component-scan base-package="xxx.xxx" />
```

后就解决了，这个还可以再深入研究下。
