---
title: Windows 10 下安装Tensorflow GPU版
date: 2018-04-29 09:41:04
tags:
- tensorflow
reward: true
brief: "test"
---

之前在自己的windows上安装了tensorflow1.0.1-CPU版，后来想用gpu进行计算，于是安装gpu版。没想到软件之间的依赖关系、版本等导致数种问题，百度谷歌良久才调通程序，特记下曲折的安装过程和一些细节，尽量解释选择软件版本的原因，希望能减轻读者的痛苦。
<!-- more -->
## 我的环境
- **操作系统：Windows 10 64bit**
- **显卡型号：GeForce GTX 950M**

## 软件准备

- **Anaconda3-4.2.0-Windows-x86_64.exe**
> <a href="https://www.continuum.io/" target="_blank">Anaconda</a>是一个用于科学计算的Python发行版，安装后，python以及常用的用于计算的package如numpy、matplotlib、Pillow也都安装好了，而且还能切换不同版本的python，很方便。
>  安装tensorflow是不必安装Anaconda的，但是你至少要有python环境，而且需要是python3.5.x，因为正式1.0版的tensorflow不支持python2.x版。
> Anaconda3中内置的python为3.x版本（Anaconda2中内置的python为2.x版本），不过最新的Anaconda内置python为3.6版，为了避免潜在的麻烦，我安装了较新的历史版本（没错，Anaconda可以切换python版本，不过我之前在Anaconda2下安装python3后出现了一些问题，没时间较真了）。
>
> <a href="https://repo.continuum.io/archive/index.html" target="_blank">Anaconda 官网历史版本下载</a>
>
> <a href="http://python.jobbole.com/86236/" target="_blank">一篇不错的入门教程</a>

- **tensorflow_gpu-1.0.1-cp35-cp35m-win_amd64.whl**
>   <a href="https://github.com/tensorflow/tensorflow" target="_blank">tensorflow</a>在github的提供了以下几个版本：</br>
    * Linux CPU-only: Python 2 / Python 3.4 / Python 3.5</br>
    * Linux GPU: Python 2 / Python 3.4 / Python 3.5</br>
    * Mac CPU-only: Python 2 / Python 3 </br>
    * Mac GPU: Python 2 / Python 3 </br>
    * Windows CPU-only: Python 3.5 64-bit</br>
    * <a href="https://ci.tensorflow.org/view/Nightly/job/nightly-win/DEVICE=gpu,OS=windows/lastSuccessfulBuild/artifact/cmake_build/tf_python/dist/tensorflow_gpu-1.0.1-cp35-cp35m-win_amd64.whl" target="_blank">Windows GPU: Python 3.5 64-bit</a></br>
    * Android: demo APK, native libs

- **DirectX SDK**
><a href="https://www.microsoft.com/en-us/download/details.aspx?id=6812" target="_blank">官网下载</a>
- **VS2015/2013/2012**
>**安装cuda前必须先安装一款VS**，具体安装哪款，请参照下面对于cuda的介绍。

- **cudnn-8.0-windows10-x64-v5.1**
>针对深度神经网络采用GPU计算的加速库，下载解压后你会得到3个.dll文件。
>
><a href="https://developer.nvidia.com/cudnn" target="_blank">官网下载（下载需要注册，并且回答一些问题）</a>

## 安装 cuda

>cuda是一种并行计算平台，可以利用gpu进行并行计算。
**请注意，tensorflow1.x-gpu版只支持cuda8.x版本**，[你可以在这里确认这个问题](https://github.com/tensorflow/tensorflow/issues/8161)

  操作系统对cuda8的支持（常用的系统应该都没问题，即使在cross情况下——32位的系统在64位机器上）:<br/>

|Operating System |Native x86_64|Cross (x86_32 on x86_64)|
|:--------:|:--------:|:--------:|
|Windows 10     |YES    |YES|
|Windows 8.1    |YES    |YES|
|Windows 7  |YES    |YES|
|Windows Server 2012 R2     |YES    |NO|
|Windows Server 2008 R2 DEPRECATED  |YES    |YES|

VS对cuda8的支持(**注意cross情况**):

|Compiler IDE|Operating System |Native x86_64|Cross (x86_32 on x86_64)|
|:-------- |:-------- |:--------:|:--------:|
|Visual C++ 14.0    |Visual Studio 2015 |   YES     |YES
|Visual C++ 14.0    |**Visual Studio Community 2015**   |YES    |**NO**
|Visual C++ 12.0    |Visual Studio 2013     |YES    |YES
|Visual C++ 11.0    |Visual Studio 2012     |YES    |YES
|Visual C++ 10.0 DEPRECATED     |Visual Studio 2010     |YES    |YES
<a href="http://docs.nvidia.com/cuda/cuda-installation-guide-microsoft-windows/index.html#compiling-examples__valid-results-from-sample-cuda-bandwidthtest-program" target="_blank">cuda官网安装教程（这里有上面两张表格）</a>
>我之前就不幸地安装了唯一一种不支持的情况，安装Visual Studio Community 2015 32bit在我64bit笔记本上，报了一堆错，卸载后安装vs2013后解决问题。<br/>到这里你就可以根据自己的情况安装对应的VS了。
>
<a href="https://developer.nvidia.com/cuda-downloads" target="_blank">官网下载</a>



-------------------

## 一些问题
- **cudnn是必要的**
我之前参考了一些教程，以为cudnn只是为了提速，并不是必须的，就没有安装，结果出错。在[tensorflow官网安装教程](https://www.tensorflow.org/install/install_windows)中，要求cudnn是必装的，并且要将动态链接库文件添加到环境变量中，我照做后仍报错，后来参照[这篇教程](http://www.jianshu.com/p/c245d46d43f0)将3个.dll文件拷贝到duda安装目录下对应路径下，解决问题。
