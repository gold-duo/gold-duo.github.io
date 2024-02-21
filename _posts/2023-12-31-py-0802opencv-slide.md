---
title: 0802使用OpenCV识别滑动验证码
date: 2023-12-31 22:30
categories: [python,网络爬虫开发实战]
tags: [python,网络爬虫开发实战,OpenCV,验证码,滑动验证码] 
---

通过边缘算法( Canny Edge Detection)获取到图片的边缘特征，再通过面积、周长、位置排查出滑动缺口（OpenCV提供的功能）。

## 准备工作
- 安装opencv-python:`pip3 install opencv-python`

## 用到的OpenCV功能：
- `cv2.GaussianBlur`高斯模糊。这里用高斯模糊的目的是去除干扰凸显边缘轮廓
    - src，输入图像，即源图像，填Mat类的对象即可。它可以是单独的任意通道数的图片， 但需要注意，图片深度应该为CV_8U,CV_16U, CV_16S, CV_32F 以及 CV_64F之一。
    - ksize，高斯内核的大小。其中ksize.width和ksize.height可以不同，但他们都必须为正数和奇数（并不能理解）。或者，它们可以是零的，它们都是由sigma计算而来。
    - sigmaX，表示高斯核函数在X方向的的标准偏差。
    - sigmaY，表示高斯核函数在Y方向的的标准偏差。若sigmaY为零，就将它设为sigmaX，如果sigmaX和sigmaY都是0，那么就由ksize.width和ksize.height计算出来。
   - 返回值： 模糊后的image
- `cv2.Canny`对图片进行边缘检测的
    - image：要检测的图像
    - threshold1：阈值1（最小值）
    - threshold2：阈值2（最大值），使用此参数进行明显的边缘检测
    - apertureSize：sobel算子（卷积核）大小
    - L2gradient ：布尔值。 True：使用更精确的L2范数进行计算（即两个方向的导数的平方和再开方）False：使用L1范数（直接将两个方向导数的绝对值相加）
    - 返回值：一组数组表示边缘
- `cv2.findContours`对边缘轮廓提取
    - image：图片
    - mode：轮廓的检索模式
        - RETR_EXTERNAL表示只检测外轮廓
        - RETR_LIST检测的轮廓不建立等级关系
        - RETR_CCOMP建立两个等级的轮廓，上面的一层为外边界，里面的一层为内孔的边界信息。如果内孔内还有一个连通物体，这物体的边界也在顶层。
        - RETR_TREE建立一个等级树结构的轮廓。
    - method：定义轮廓的近似方法
        - CHAIN_APPROX_NONE存储所有的轮廓点，相邻的两个点的像素位置差不超过1，即max（abs（x1-x2），abs（y2-y1））==1
        - CHAIN_APPROX_SIMPLE压缩水平方向，垂直方向，对角线方向的元素，只保留该方向的终点坐标，例如一个矩形轮廓只需4个点来保存轮廓信息
        - CHAIN_APPROX_TC89_L1，CV_CHAIN_APPROX_TC89_KCOS使用teh-Chinl chain 近似算法
    - 返回两个值，一个是轮廓本身，还有一个是每条轮廓对应的属性。
- `cv2.boundingRect`用于计算包围给定点集或轮廓的最小矩形的函数。
    - 输入参数:点集或轮廓
    - 返回值： 包围输入点集或轮廓的最小矩形(x, y,width,height)。
- `cv2.contourArea`计算轮廓面积
    - contour:轮廓
    - oriented:方向标识（s顺时针还是逆时针，True/False）
- `cv2.arcLength`计算轮廓的周长
    - curve:轮廓
    - closed:轮廓是否封闭
## 同类算法
- [链接1](https://github.com/sml2h3/ddddocr/blob/master/ddddocr/__init__.py#L2656C9-L2656C20)

