---
title: 0804使用深度学习识别滑动验证码的缺口
date: 2024-03-09 21:00
categories: [python,网络爬虫开发实战]
tags: [python,网络爬虫开发实战,深度学习,验证码,滑动验证码,deeplearn] 
---

通过目标检测算法YOLO(You Only Look Once)实现对滑动验证码缺口的识别。（跟上节一样作者也是拿github中的开源项目来充数😄）


## 一些概念
### 市面上的目标检测算法

R-CNN,FaSt R-CNN、FasterR-CNN、SSD、YOLO

### 两种实现目标检测算法
- 一.One Stage:不需要产生候选框，直接将目标的定位和分类问题转化为回归问题，俗称“看一眼”，使用这种实现的算法有 YOLO 和 SSD，这种方法虽然正确率不及 Two Stage，但架构相对简单，检测速度更快。
- 二.Two Stage:算法首先生成一系列目标所在位置的候选框，再对这些框选出来的结果进行样本分类，即先找出来在哪儿，再分出来是啥，俗称“看两眼”，使用这种实现的算法有R-CNNFast R-CNN、Faster R-CNN 等，这种方法架构相对复杂，但正确率高。

## 开工
### 一.训练数据来源

直接爬 https://captcha1.scrape.center/ 一千次把缺口验证码保存下来

### 二.标注工具：[labellms](https://github.com/tzutalin/labellms)。这个工具用于手动对每一张训练用到的缺口验证码图标注验证码的缺口
- 安装:`pip3 install labelImg`
- 运行：命令行`labelImg`

### 三.训练
标注完就是训练了`train.sh`

### 四.测试训练效果`detect.sh`