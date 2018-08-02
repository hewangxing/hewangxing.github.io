---
title: 使用Anaconda管理python开发环境
comments: true
date: 2018-05-29 21:34:41
categories: 编程
tags: python
---
　　这篇博客用来记录Udacity无人驾驶课程的环境搭建，以减少不必要的重复劳动。
　　参考链接：
[Configure and Manage Your Environment with Anaconda][1]
[Anaconda and Jupyter Notebooks][2]
### Windows系统
　　我的系统是win10 64位的，所以下载Windows-x86_64版的[miniconda][3]，安装成功后打开**Anaconda Prompt**。接下来就可以搭建carnd-term1环境了：

    git clone https://github.com/udacity/CarND-Term1-Starter-Kit.git
    cd CarND-Term1-Starter-Kit

进入CarND-Term1-Starter-Kit目录后，将meta_windows_patch.yml重命名为meta.yml。然后执行

    conda env create -f environment.yml
就可以按照environment.yml文件创建carnd-term1环境，用conda info --envs可以查看是否创建成功，如果你需要卸载这个环境，执行：

    conda env remove -n carnd-term1
    
之后我们需要运行一些代码来测试：

    git clone https://github.com/udacity/CarND-Term1-Starter-Kit-Test.git
    cd CarND-Term1-Starter-Kit-Test
进入carnd-term1环境并运行jupyter notebook来验证安装的包是否可以正常工作

    activate carnd-term1
    jupyter notebook test.ipynb
浏览器会自动跳转到 http://localhost:8888/notebooks/test.ipynb 网址，我在测试中发现缺少requests模块, 所以就手动安装：

    conda install requests

解决了这个问题，经测试，jupyter notebook的每个单元都能够正常执行，没有错误，说明Windows环境下的carnd-term1已经完全ok了。效果图如下：
{% asset_img 20180529222441.png  jupyter验证成功界面 %}

  [1]: https://github.com/udacity/CarND-Term1-Starter-Kit/blob/master/doc/configure_via_anaconda.md
  [2]: https://classroom.udacity.com/courses/ud1111
  [3]: https://repo.continuum.io/miniconda/Miniconda3-latest-Windows-x86_64.exe