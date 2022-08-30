+++
author = "H. Wang"
title = "ROS Python3"
date = "2021-09-20"
description = "在Python3中使用ROS(cv_bridge和tf)"
tags = [
    "ROS",
    "Python"
]
categories = [
    "ROS"
]

image = "python3-ros.png"
+++

## 错误介绍
### 在Python3中使用ROS会出现cv_bridge和tf报错问题
[python 3.x - Unable to use cv_bridge with ROS Kinetic and Python3 - Stack Overflow](https://stackoverflow.com/questions/49221565/unable-to-use-cv-bridge-with-ros-kinetic-and-python3?rq=1)

**python3 cv bridge报错**
```
ImportError: dynamic module does not define module export function (PyInit_cv_bridge_boost)
```

**tf tf2报错**
```
ImportError: dynamic module does not define module export function (PyInit__tf2)
```
解决的关键在于使用python3编译，可以通过下面参数进行cmake
```sh
-DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.6m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.6m.so
```
这里介绍常用的catkin工具进行编译

### ROS Wiki中对于软件包的介绍
- [ROS tf Package](http://wiki.ros.org/tf)
- [ROS cv_bridge Package](https://wiki.ros.org/cv_bridge)

## CV Bridge
### 依赖
> + `python-catkin-tools` catkin工具
> + `python3-dev` 和 `python3-catkin-pkg-modules` 用于编译
> + `python3-numpy` 和 `python3-yaml` cv_bridge依赖
> + `ros-melodic-cv-bridge` 通过 cv_bridge ros 包来安装和cv_bridge相关的依赖，在安装ROS过程中可能已经安装
```sh
sudo apt-get install python-catkin-tools python3-dev python3-catkin-pkg-modules python3-numpy python3-yaml ros-melodic-cv-bridge
```
### 创建catkin工作空间
```sh
mkdir -p cvbdg_ws/src
```
```sh
cd cvbdg_ws
catkin init
```
### 配置catkin编译参数
设置使用python3编译
```
catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.6m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.6m.so
```
### 设置编译安装位置
位于$CATKIN_WORKSPACE/install 目录下
```
catkin config --install
```
### clone cv_bridge 源码
```
git clone https://github.com/ros-perception/vision_opencv.git src/vision_opencv
```
### 检查版本
```
apt-cache show ros-kinetic-cv-bridge | grep Version
# 结果
Version: 1.13.0-0bionic.20210505.0322380
```
### 检查git仓库中是否有对应版本并检出
```sh
cd src/vision_opencv/
git checkout 1.13.0
#返回工作空间主目录
cd ../../
```
### 编译cv bridge
```sh
catkin build cv_bridge
```
### 设置环境变量
使用 `--extend` 命令source附加环境变量
```sh
source install/setup.bash --extend
```
因为每次使用都需要这样的`source`操作，可以将其加入 `.bashrc` 中打开终端自动执行
```sh
echo "source $HOME/cvbdg_ws/install/setup.bash --extend" >> ~/.bashrc
```
### 测试
```sh
$ python3

Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> from cv_bridge.boost.cv_bridge_boost import getCvType
>>>
```
如果没有再出现错误即编译安装成功

## TF TF2
### 依赖
和安装编译cv bridge相同，安装部分依赖
```sh
sudo apt-get install python-catkin-tools python3-dev python3-catkin-pkg-modules python3-numpy python3-yaml ros-melodic-geometry ros-melodic-geometry2
```
创建catkin工作空间
```sh
mkdir -p tf_ws/src
```
```sh
cd tf_ws
catkin init
```
### 配置catkin编译参数
设置使用python3编译
```sh
catkin config -DPYTHON_EXECUTABLE=/usr/bin/python3 -DPYTHON_INCLUDE_DIR=/usr/include/python3.6m -DPYTHON_LIBRARY=/usr/lib/x86_64-linux-gnu/libpython3.6m.so
```
### 设置编译安装位置
位于$CATKIN_WORKSPACE/install 目录下
```sh
catkin config --install
```
### clone 源码
```sh
git clone -b melodic-devel https://github.com/ros/geometry
git clone -b melodic-devel https://github.com/ros/geometry2
```
### 检查版本
检查系统中安装的软件包版本
```sh
apt-cache show ros-melodic-geometry | grep Version
# 结果
Version: 1.12.1-1bionic.20210505.035349
```
### 编译tf tf2
```sh
catkin build
```
### 环境变量
使用 `--extend` 命令source附加环境变量
```sh
source install/setup.bash --extend
```
加入 .bashrc 中打开终端自动执行
```sh
echo "source $HOME/tf_ws/install/setup.bash --extend" >> ~/.bashrc
```
### 测试
```sh
$ python3

Python 3.5.2 (default, Nov 23 2017, 16:37:01)
[GCC 5.4.0 20160609] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> import tf
>>>
```
如果没有再出现错误即编译安装成功

> 另外，可以通过源码编译的方式安装ROS，使ROS支持Python3
如：[Building ROS Melodic with Python3 support](https://www.miguelalonsojr.com/blog/robotics/ros/python3/2019/08/20/ros-melodic-python-3-build.html)
