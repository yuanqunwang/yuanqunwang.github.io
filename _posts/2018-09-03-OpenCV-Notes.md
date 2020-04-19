---
title: OpenCV Setup
tags: [OpenCV, Tools]
---
## OpenCV环境配置
### Visual Studio开发配置
1.下载opencv安装文件，解压缩至`C:\opencv`
2.设置头文件包含目录：调试-XX属性-VC++目录-包含目录（`C:\opencv\build\include`)
3.设置库目录：调试-XX属性-VC++目录-库目录（`C:\opencv\build\x64\vc15\lib`)
4.设置连接器：调试-XX属性-链接器-输入-附加依赖库（`opencv_world342d.lib`)

### Mac环境下XCode开发配置

1. 通过homebrew安装OpenCV：`brew install opencv`
2. 设置项目头文件查找目录(Search Paths-Header Search Paths)：`/usr/local/Cellar/opencv/3.4.2/include`
3. 设置项目库目录(Search Paths-Library Search Paths)：`/usr/local/Cellar/opencv/3.4.2/lib`
4. 设置项目链接器(Linking-Other Linker Flags)Flags：`pkg-config --cflags --libs opencv`命令的输出内容


## OpenCV基础
### 图像的基本操作
* OpenCV中的Mat类代表一个n维（n>2）的单通道或多通道数组，可以存储向量、矩阵、灰度图、彩色图。
