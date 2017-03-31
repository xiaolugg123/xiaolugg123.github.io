---
layout: post
title:  "MACOS PRO 编译 android x84_64 内核"
date:   2017-03-31
excerpt: "编译环境配置 文件替换"
tag:
- 编译环境 
- 文件替换
- 编译命令

---
# MACOS PRO 编译 android x84_64 内核
编译环境配置 文件替换

### 编译环境 


### 编译命令
macos 下需要安装两个工具
`brew install gnu-sed coreutils`
不安装 编译完成的BzImage不能使用
安装完成使用 `brew info gnu-sed coreutils` 查看变量: 

```
export PATH="/usr/local/opt/gnu-sed/libexec/gnubin:$PATH" 
export PATH="/usr/local/opt/coreutils/libexec/gnubin:$PATH"
export ARCH=x86_64
export CROSS_COMPILE=x86_64-linux-android-
cd kernel(内核根目录)
make fugu_defconfig
make
```


