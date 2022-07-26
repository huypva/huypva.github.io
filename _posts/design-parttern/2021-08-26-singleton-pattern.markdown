---
layout: post
title: Singleton Pattern 
date: 2021-08-26 10:00:20 +0700
description: Pattern tạo ra một instance duy nhất của một class
img: design_pattern/singleton.png
tags: [String, Java]
categories: [Java]
---

- *Singleton* là pattern tạo ra một instance duy nhất của một class

{% highlight shell %}
public class Singleton {

  private static final Singleton instance = new Singleton();

  public Singleton() {
    //Do something in constructor
  }

  public static Singleton getInstance() {
    
    return instance;
  }
}
{% endhighlight %}

- *Lazy singleton* tạo instance khi được sử dụng

{% highlight shell %}
public class LazySingleton {

  private static LazySingleton instance;

  public LazySingleton() {
    //Do something in constructor
  }

  public static LazySingleton getInstance() {
    if (instance == null) {
      instance = new LazySingleton();
    }

    return instance;
  }
}
{% endhighlight %}

- *Double checked singleton* đảm bảo tạo singleton trong môi trường mutilthread

{% highlight shell %}
public class DoubleCheckedSingleton {

  private static DoubleCheckedSingleton instance;

  public DoubleCheckedSingleton() {
    //Do something in constructor
  }

  public static DoubleCheckedSingleton getInstance() {
    if (instance == null) {
      synchronized (DoubleCheckedSingleton.class) {
        if (instance != null) {
          instance = new DoubleCheckedSingleton();
        }
      }
    }

    return instance;
  }
}
{% endhighlight %}

- *BillPugh singleton* tạo singleton bằng cách sử dụng static inner class

{% highlight java %}
public class BillPughSingleton {

  private BillPughSingleton(){}

  private static class SingletonHelper {

    private static final BillPughSingleton INSTANCE = new BillPughSingleton();
  }

  public static BillPughSingleton getInstance() {

    return SingletonHelper.INSTANCE;
  }
}
{% endhighlight %}