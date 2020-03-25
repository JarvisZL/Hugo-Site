---
author: "Lingyu Zhang"
author_link: "https://jarviszly.com"
title: "Android UI控件Textview简介"
date: 2020-03-24T22:40:56+08:00
lastmod: 2020-03-24T22:40:56+08:00
draft: false
description: ""
show_in_homepage: true
description_as_summary: false
license: ""

tags: [Android]
categories: []

featured_image: ""
featured_image_preview: ""

comment: true
toc: true
auto_collapse_toc: true
math: true
---

Android :(fab fa-android): UI控件Textview简单介绍。
<!--more-->

## 基本属性介绍
最基本的几个属性略。
- textColor: 字体颜色; textStyle: 字体风格; background: 背景颜色(可以使用图片)
```xml
 <TextView
    android:id="@+id/txtOne"
    android:layout_width="200dp"
    android:layout_height="200dp"
    android:gravity="center"
    android:text="TextView"
    android:textColor="#EA5246"
    android:textStyle="bold|italic"
    android:background="#000000"
    android:textSize="18sp" 
/>
```

## 其他属性介绍

### 阴影相关属性
- shadowColor: 阴影颜色；shadowRadius: 阴影模糊程度；shadowDx,shadowDy: 阴影在水平和竖直方向上的偏移。
```xml
 <TextView
    ...
    android:shadowColor="#F9F900"
    android:shadowDx="10.0"
    android:shadowDy="10.0"
    android:shadowRadius="3.0"
    ...
/>
```

### 带图片的Textview
- 核心时使用**drawableXXX**
  - drawableTop(上),drawableButtom(下),drawableLeft(左),drawableRight(右),drawablePadding来设置图片与文字间的间距。
```xml
<TextView  
    ... 
    android:drawableTop="@drawable/p1"  
    android:drawableLeft="@drawable/p2"  
    android:drawableRight="@drawable/p3"  
    android:drawableBottom="@drawable/p4"  
    android:drawablePadding="5dp"  
    android:text="Hello,world." 
/>  
```
- 单纯在xml文件里面使用，无法自行设置图片大小，这时候需要在java代码中进行修改
```java
textview = (TextView) findViewById(R.id.txt);  
Drawable[] drawable = textview.getCompoundDrawables();  
//drawable[1]x轴为(100,200),y轴为(0,200)
drawable[1].setBounds(100, 0, 200, 200);  
// 数组下表0~3,依次是:左上右下
textview.setCompoundDrawables(drawable[0], drawable[1], drawable[2],  
        drawable[3]); 
```

### 连接跳转
- 当textview.text是链接，邮件，电话等，可以通过 **autoLink**属性实现跳转,其取值可选web,email,phone,map,all。
```xml
<TextView  
    ...
    android:text="https://jarviszly.com"
    android:autoLink="web"
/>  
```
或者不在xml文件里面写，而在java文件里面使用
```java
textview.setAutoLinkMask(Linkify.ALL);
textview.setMovementMethod(LinkMovementMethod.getInstance());//否则点击无响应。
```
### Html

```java
TextView textview = (TextView)findViewById(R.id.txt);
String s1 = "<font color='blue'><b>沿窗听风尘，痕迹都黯然。</b></font><br>";
s1 += "<a href = 'https://jarviszly.com'>浙语临湖</a>";
textview.setText(Html.fromHtml(s1));
textview.setMovementMethod(LinkMovementMethod.getInstance());
```

### SpannableString定制
- SpannableString可以对文本内容进行定制，如颜色，字体，链接等
- 详细情况请点击：[TextView(文本框)详解](https://www.runoob.com/w3cnote/android-tutorial-textview.html)