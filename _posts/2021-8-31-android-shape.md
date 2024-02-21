---
title: shape模版
date: 2021-8-31 23:24
categories: [android] 
tags: [android,shape] 
---

```xml
<?xml version="1.0" encoding="utf-8"?>
<shape  
      xmlns:android="http://schemas.android.com/apk/res/android"
      android:shape=["rectangle" | "oval" | "line" | "ring"] >   <!--默认为rectangle-->
      <corners  <!-- shape=“rectangle”时使用， -->
            android:radius="integer"  <!-- 半径，会被下边的属性覆盖，默认为1dp，-->
            android:topLeftRadius="integer" 
            android:topRightRadius="integer"
            android:bottomLeftRadius="integer"
            android:bottomRightRadius="integer" />
      <gradient  <!-- 渐变-->
            android:angle="integer"
            android:centerX="integer"
            android:centerY="integer"
            android:centerColor="integer"
            android:endColor="color"
            android:gradientRadius="integer"
            android:startColor="color"
            android:type=["linear" | "radial" | "sweep"]
            android:useLevel=["true" | "false"] />
      <padding
            android:left="integer"
            android:top="integer"
            android:right="integer"
            android:bottom="integer" />
      <size    <!--指定大小，一般用在imageview配合scaleType属性使用。大小一般会适配滴-->
            android:width="integer"
            android:height="integer" />
      <solid    <!--填充颜色，可是是十六进制颜色。（比如想设置半透明效果，直接使用十六就只就OK）-->
            android:color="color" />
      <stroke    <!-- 指定边框，border，dashWidth和dashGap有一个为0dp则为-->
            android:width="integer"
            android:color="color"
            android:dashWidth="integer"    <!-- 虚线宽度-->
            android:dashGap="integer" />    <!-- 虚线间隔宽度-->
</shape>
```

