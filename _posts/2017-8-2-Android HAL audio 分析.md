---
layout: post
title:  "Android HAL audio 硬件 audio_hw.c 的分析"
date:   2017-08-02
excerpt: "android audio HAL"
tag:
- HAL 简单介绍 
- 
- 
---
# Android HAL 介绍
Android 系统硬件抽象层（Hardware Abstract Layer）运行在用户空间中，它向下屏蔽了硬件驱动的实现细节，向上提供了硬件访问的服务。通过 HAL 层，Android 系统分为两层来支持硬件设备，其中一层实现在用户空间，另外一层实现在内核空间中。
# HAL 层的作用
Android 在用户空间中新建一个的 HAL 层来支持硬件设备的主要原因还是因为 Android 使用的开源协议是 Apache License，这个协议比较宽松，它允许开发者获取并修改了源码之后，不用把源码公开出来。而 Linux 使用的开源协议 GPL，它的要求限制就比较多，它要求开发者添加或修改了源码之后，必须把添加或修改后的代码公开出来，所以我们在 Linux 内核中的所使用的设备驱动程序都是源码公开的，任何人都可以获取并修改它。

因此，如果 Android 系统像其他 Linux 系统一样，把对硬件的支持完全实现在 Linux 内核的驱动模块中，那么硬件厂商就必须将这些硬件驱动源码公开，这样就可能损害到移动厂商的利益，因为这相当于暴露了硬件的实现细节和参数。

所以，Android 就在用户空间搞了一个 HAL 层，将硬件的一些重要的操作都放在这一层中完成，这些操作都封装在厂商所提供的一个动态链接库中，从而达到了避免源码公开的目的，而底层 Linux 内核空间中的设备驱动模块，现在则只提供一些最基本的硬件设备寄存器操作的功能。
# HAL 层 模块实现
由于最近一直在做一个 ANDROID TV 系统相关的项目，主要就是在HAL 层构建一个音频通道，所以我就重新对 HAL 的相关知识进行了一个学习和整理，所以本文中将以音频系统对应的 audio HAL 模块（ 它最终是以 `audio.primary.omni.so` 的动态链接库形式存在 ）为例来介绍一个 HAL 模块的定义及实现过程。
# HAL 层的三个重要结构体
Android 系统的 HAL 层其实并不复杂，只要你能理解清楚下面这 3 个结构体的含义：

 **hw_module_t**：用来描述硬件模块  
 **hw_device_t**：用来描述硬件设备  
 **hw_module_methods_t**：用来打开硬件模块中包含硬件设备，获得指向硬件设备结构体的指针  
Android 系统中 HAL 层是以模块的方式来管理各个硬件访问的接口，每一个硬件模块都对应一个动态链接库文件，而这些动态链接库文件需要符号一定的规范，而上述的这 3 种结构体就是用来建立这种规范。并且一个硬件模块可以管理多个硬件设备，例如 audio HAL 硬件模块中就管理了扬声器、麦克风等多个硬件设备。

**注意：这里一定区分 __hw_module_t__ 和 __hw_device_t__ 它们所表示的含义**
# Audio HAL 模块的实现
## Step 1：定义 struct audio_module 模块
我们前面在结构体 hw_module_t 介绍时，有提到具体的硬件模块要定义一个新的结构体并且这个结构体的==第一个成员必须是 hw_module_t 类型==，所以根据这个规则，audio_module 的定义如下所示： 代码路径：`/hardware/libhardware/include/hardware/audio.h`
```c
/**
 * Name of the hal_module_info
 */
#define HAL_MODULE_INFO_SYM         HMI

/**
 * Name of the hal_module_info as a string
 */
#define HAL_MODULE_INFO_SYM_AS_STR  "HMI"
```
而 struct audio_module 类型名为 HAL_MODULE_INFO_SYM 变量的定义如下所示： 代码路径：`../hardware/libhardware/modules/audio/audio_hw.c`
```c
struct audio_module HAL_MODULE_INFO_SYM = {
    .common = {
        .tag = HARDWARE_MODULE_TAG,
        .module_api_version = AUDIO_MODULE_API_VERSION_0_1,
        .hal_api_version = HARDWARE_HAL_API_VERSION,
        .id = AUDIO_HARDWARE_MODULE_ID,
        .name = "Default audio HW HAL",
        .author = "The Android Open Source Project",
        .methods = &hal_module_methods,
    },
};
```
# 创建文件
在`/device/<compary>/<device_name>/audio`目录  
目录下包含**audio_hw.c Android.mk**  
目前包含这两个，后续androids_procy.c 是否需要，目前没有研究清楚。
使用mm  编译，编译后在 `out/target/product/flounder/obj_arm/lib`
下会有.so生成
#audio_hw.c 分析
```c
struct stub_audio_device {
    struct audio_hw_device device;
};

struct stub_stream_out {
    struct audio_stream_out stream;
    int64_t last_write_time_us;
};

struct stub_stream_in {
    struct audio_stream_in stream;
    int64_t last_read_time_us;
};
```
*** 分析***

```c
static uint32_t out_get_sample_rate(const struct audio_stream *stream)
{
    return 16000;
}
```
__分析__
## rate 设置为16000
使用 16×16000 输入
#权限 打开/dev/hidraw0