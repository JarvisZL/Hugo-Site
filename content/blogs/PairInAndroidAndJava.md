---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "Pair In Android And Java"
date: 2020-03-25T10:45:04+08:00
lastmod: 2020-03-25T10:45:04+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: ["Android","java"]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---


Pair在Android:(fab fa-android):和Java:(fab fa-java):中的异同点。
<!--more-->

## Pair In Android
Android里面的Pair在包[android.util](https://developer.android.com/reference/android/util/package-summary)中，类名为<font color= "#2471A3 " size=3>android.util.Pair<F,S></font>,其继承自类<font color= "#2471A3 " size=3>java.lang.Object</font>.
<br>

```java
//Fields, 均为public,所以可以直接对象.first/second获得
public final F first
public final S second
//Constructors
Pair(F first, S second)
e.g. Pair p1 = new Pair<Integer,String>(0,"hello");
// Public Methods
static <A,B> Pair<A,B> creat(A a,B b)//实际上内部调用构造函数完成。
e.g. Pair p2 = Pair.creat(1,"world");

boolean equals(Object o)
int hashCode()
String toString
```

## Pair In java
Java里面的Pair包同样继承自<font color= "#2471A3 " size=3>java.lang.Object</font>，其所在的包为[javafx.util](https://docs.oracle.com/javase/9/docs/api/javafx/util/package-summary.html)。
<br>

```java
//Fileds
private K key
private V value

//Constructors
Pair(K key, V value)
e.g. Pair<Integer,String> p1 = new Pair<>(0,"hello");

//Instance Methods
boolean equals(Object o)
K getKey()
V getValue()
int hashCode()
String toString()
```