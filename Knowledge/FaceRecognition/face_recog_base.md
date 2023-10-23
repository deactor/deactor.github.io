---
title:  "人脸识别基础原理"
---
### 人脸识别基本流程
人脸注册：  
人脸检测->图像处理->特征提取->保存特征信息到人脸库。  
人脸比对时：  
人脸检测->图像处理->特征提取->将特征信息到人脸库中进行匹配。  

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_recog_process.png)

问题：人脸的检测是基于什么原理？  
基于机器学习，首先是提供大量的人脸图片，标注出人脸数据供机器学习，最终得到一个人脸模型。当再给机器一个未标注的人脸图片时，机器就可以根据之前学习到的人脸模型来自己判断人脸。  

### 深度信息
正常的2D识别，特征提取的信息包含特征的长度和宽度。这种识别无法防止图片假冒。  
3D识别，特征信息中包含一个深度信息，即人脸特征点到相机的垂直距离。就可以防止图片假冒，也即活体检测。  

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_deep_info.png)

## 深度信息获取的3种方式：
### 1.ToF飞行时间技术
TOF深度相机，包含光源和光电响应模块。  
光源发出激光（如红外激光），并接收反射回来的激光，传感器通过发射和反射光之间的时间换算出到目标的距离，即深度信息。  

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_fly_time.png)

### 2.双目测距
1. 两个RGB摄像头进行拍摄得到两个不一样的平面图像
2. 标注两个图像上的特征点
3. 基于三角测量原理间接计算出深度信息

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_dual_instance.png)

难点在于如何准确标注出两幅图像的共同特征点来。  
问题：是怎么根据三角测量原理计算的？

### 3.结构光：
1. 点阵投影器投影光点到人脸上。
2. 红外摄像头捕捉投影到脸上的光点，形成红外图像。
3. 通过三角测量原理计算深度信息。

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_struct_light.png)

结构光分类：
+ 散斑结构光：光点带有随机性，安全性好，计算量大。
+ 编码结构光：经过编码的固定光点，计算量小，但是安全性比前者差。

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_struct_light_type.png)

之后需要将深度图像与平面RGB图像进行结合，得到三维人脸信息。  
问题：为什么要将深度图和RGB图像进行结合？  
猜测：是因为特征点是在RGB图像上标注的，需要叠加RGB图像与深度图像，来得到特征点的深度数据吗？那这样的话，RGB图像与深度图的分辨率不同不会造成特征点深度数据有误吗？  
答案：猜测基本正确，对于RGB图像与深度图分辨率不同的问题，他们在进行匹配前是需要经过“配准”的，以保证他们的像素点一一对应。

![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_deep_match_1.png)
![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_deep_match2.png)

三种方案对比：  
![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_recog_compare.png)

### 活体检测：
![icon](https://raw.githubusercontent.com/deactor/deactor.github.io/master/imgs/face_live_recog_type.png)

### 参考：
youtube视频  
http://www.vision263.com/2084.html  
https://blog.csdn.net/liuxiao214/article/details/85485155