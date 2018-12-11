---
layout: post
title: "Java 查漏补缺之：ThreadLocal 使用"
tagline: ""
description: ""
category: 学习笔记
tags: [java, thread,]
last_updated:
---

ThreadLocal 线程本地变量，每个线程保存变量的副本，对副本的改动，对其他的线程而言是透明的。

## 常用方法

    // 设置当前线程的线程局部变量的值
    void set(Object value);

    // 该方法返回当前线程所对应的线程局部变量
    public Object get();

    // 将当前线程局部变量的值删除
    public void remove();


ThreadLocal 类允许我们创建只能被同一个线程读写的变量，通常的用法是当有一些 Object 不是线程安全，但是又想避免使用[同步](https://docs.oracle.com/javase/tutorial/essential/concurrency/sync.html) 访问机制时。比如 SimpleDateFormat，因此可以使用 ThreadLocal 来给每一个线程提供一个线程自己的对象。

    public class Foo
    {
        // SimpleDateFormat is not thread-safe, so give one to each thread
        private static final ThreadLocal<SimpleDateFormat> formatter = new ThreadLocal<SimpleDateFormat>(){
            @Override
            protected SimpleDateFormat initialValue()
            {
                return new SimpleDateFormat("yyyy-MM-dd HH:mm:ss");
            }
        };

        public String formatIt(Date date)
        {
            return formatter.get().format(date);
        }
    }

ThreadLocal 对象是给定线程中对象的引用，因此在服务端使用线程池时极有可能造成 classloading leaks 内存泄露 ，在使用时需要特别注意清理。ThreadLocal 持有的任何对象空间在 Java 永久堆上，即使重新部署 webapp 也不回收这部分内存，可能造成 java.lang.OutOfMemoryError 异常。

## 原理
利用 HashMap，在 map 中保存了 Thread -> 线程变量的关系，因此在多线程之间是隔离的，但同时也耗费了内存空间。

## reference

- <https://stackoverflow.com/a/817926/1820217>