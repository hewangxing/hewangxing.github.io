---
title: 让你的手机摄像头成为jupyter的眼睛
comments: true
date: 2018-07-01 10:40:58
categories: 编程
tags:
- 图像处理
- python

---

　　摄像头作为视觉的入口，无论学习图像处理还是视频聊天都必不可少。本着物尽其用的原则，我决定把我的手机摄像头开放给我的台式主机使用，主要是希望能够在jupyter上访问用来做图像处理。经过在网上的一番搜索和我自己的摸索实践，找到了一个行之有效的方案。
　　
### DroidCam
　　DroidCam可以把你的安卓设备虚拟成无线的网络摄像头，方便PC端的QQ、Skype等软件视频聊天的时候使用。也可以通过浏览器输入IP的方式来访问，十分方便易用。到Play商店搜索DroidCam Wireless Webcam可以下载安卓客户端，到官网Dev47Apps可以下载Windows客户端DroidCam.Client.6.0.FullOffline.exe。
　　首先要确保PC和手机在同一个wifi网络下再打开PC客户端和app，然后把app上显示的IP地址和端口号（默认4747，可以在Setting中修改）填入PC客户端，就可以点击Start/Connect连接了。
{% asset_img help.png 通过wifi连接 %}

　　还可以在web浏览器端输入http://...URL来访问。
{% asset_img help2.png 通过浏览器访问 %}

### jupyter访问摄像头
#### 通过OpenCV的VideoCapture类来实现
```Python
    import cv2

    # 注意将xxx换成你app上的IP地址
    video_path = 'http://xxx:4747/video'
    cap = cv2.VideoCapture(video_path)
    # cap = cv2.VideoCapture(0) 
    while(cap.isOpened()):
        ret, frame = cap.read()
        cv2.imshow('frame',frame)       
        if cv2.waitKey(1) &0xFF == ord('q'):
            break
    cap.release()
    cv2.destroyAllWindows()
```
　　上述程序在视频显示窗口中按下q键即可断开视频传输。好处是很容易编程进行图像处理，缺点是视频窗口单独显示，使用起来不太方便。
{% asset_img 1.PNG OpenCV窗口显示效果图 %}

#### 通过HTML直接内嵌在网页中
```Python
from IPython.display import HTML

# 注意将xxx换成你app上的IP地址
HTML("""
<body>
    <img style="-webkit-user-select: none;" src="http://xxx:4747/video" width="389" height="291">
</body>
""")
```
　　好处是视频图像可以直接在网页上实时显示，缺点是不方便编程进行图像处理。
{% asset_img 2.PNG HTML网页显示效果图 %}

### 牛刀小试
　　我用的是HMD出的Nokia7的前置500万像素摄像头，为了试试这个手机版的网络摄像头好不好使，我决定用Haar Cascades检测人脸和眼睛的代码来测试一下。需要将[haarcascade_frontalface_default.xml][1]和[haarcascade_eye_tree_eyeglasses.xml][2]这两个文件下载到工程根目录下，代码实现比较简单，就不细说了，如下：
```Python
    import cv2
    import numpy as np

    face_cascade = cv2.CascadeClassifier('haarcascade_frontalface_default.xml')
    eye_cascade = cv2.CascadeClassifier('haarcascade_eye_tree_eyeglasses.xml')
    # 注意将xxx换成你app上的IP地址
    video_path = 'http://xxx:4747/video'
    # cap = cv2.VideoCapture(0)
    cap = cv2.VideoCapture(video_path)
    while(cap.isOpened()):
    
        ret, img = cap.read()
        gray = cv2.cvtColor(img, cv2.COLOR_BGR2GRAY)
        faces = face_cascade.detectMultiScale(gray)
        for (x, y, w, h) in faces:
            cv2.rectangle(img, (x, y), (x+w, y+h), (255, 0, 0), 2)
            roi_gray = gray[y:y+h, x:x+w]
            roi_color = img[y:y+h, x:x+w]
            eyes = eye_cascade.detectMultiScale(roi_gray)
            for (ex, ey, ew, eh) in eyes:
                cv2.rectangle(roi_color, (ex, ey), (ex+ew, ey+eh), (0, 255, 0), 2)
            
        cv2.imshow('img',img)       
        if cv2.waitKey(1) &0xFF == ord('q'):
            break
        
    cap.release()
    cv2.destroyAllWindows()
```
{% asset_img 3.PNG 人脸检测实验效果图 %}

　　可以看到，用手机摄像头来捕捉人脸和眼睛的效果还是不错的，虽然实验过程中会有点网络延迟，但是总体影响不大，用来替代市面上的一些网络摄像头还是可行的，哈哈！
　　
### 总结
　　这次实验给我的启发：可以在手机端运行一个web服务器，并让这个web服务器来管理手机各类硬件资源，这样只要通过浏览器jupyter就可以直接访问了！唯一的问题就是要特别注意网络安全问题，最好可以设置下账号密码！


  [1]: https://github.com/opencv/opencv/blob/master/data/haarcascades/haarcascade_frontalface_default.xml
  [2]: https://github.com/opencv/opencv/blob/master/data/haarcascades/haarcascade_eye_tree_eyeglasses.xml