---
layout: post
title: "初探Java8新特性之lambda表达式"
description: ""
category: 
tags: []
---
{% include JB/setup %}


Java8带有Lambda表达式的预览版的JDK已经放出来了（地址在最下面），新特性有以下四个：

1.Lambda表达式（或称之为“闭包”或者“匿名函数”）

2.扩展的目标类型

3.方法和构造器引用

4.接口默认方法

本文先介绍一下Java8中很值得期待的Lambda表达式，lambda表达式，等同于大多说动态语言中常见的闭包、匿名函数的概念。其实这个概念并不是多么新鲜的技术，在C语言中的概念类似于一个函数指针，这个指针可以作为一个参数传递到另外一个函数中。由于Java是相对较为面向对象的语言，一个Java对象中可以包含属性和方法（函数），方法（函数）不能孤立于对象单独存在。这样就产生了一个问题，有时候需要把一个方法（函数）作为参数传到另外一个方法中的时候（比如回调功能），就需要创建一个包含这个方法的接口，传递的时候传递这个接口的实现类，一般是用匿名内部类的方式来。如下面代码，首先创建一个Runnable的接口，在构造Thread时，创建一个Runnable的匿名内部类作为参数：
<pre>
<code>
new Thread(new Runnable() {  
    public void run() {  
        System.out.println("hello");  
    }  
}).start();
</code>
</pre>
类似这种情况的还有swing中button等控件的监听器，如下面代码所示，创建该接口的一个匿名内部类实例作为参数传递到button的addActionListener方法中。
{% highlight ruby %}

public interface ActionListener {   
    void actionPerformed(ActionEvent e);  
}  

button.addActionListener(new ActionListener() {   
  public void actionPerformed(ActionEvent e) {   
    ui.dazzle(e.getModifiers());  
  }  
});

{% endhighlight %}


