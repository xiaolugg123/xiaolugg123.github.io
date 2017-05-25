---
layout: post
title:  "GLB 和 DBUS 的基本知识"
date:   2017-05-25
excerpt: "glib gio dbus bluez"
tag:
-  概论
-  DBUS
-  Bluez

---
# 概论
最近在做一个蓝牙项目，需要使用bluez的DBUS接口和app进行通讯  
本次开发基于glib-2.0 gio和GTK+3.0 开发。  
开发环境 ：
PC： DEEPIN 15.6  
嵌入式设备： raspberry pi 3b  
BLE设备： 基于TICC2650 的遥控器  
BLE 版本 BLE4.0  
主要实现功能：通过BLE的HID profile 进行数据传输，如语音，文件，以及一些特殊数据。
# Dbus 
## 通信概念
DBus通信是IPC通信机制的一种方式，它有两种模式，分别为：session(会话模式)、system(总线模式)。

总线模式：采用总线模式时，系统需要维护一个DBus Daemon，每个DBus的请求通过DBus Daemon转发。这种模式Server只需要维护一个与DBus Daemon的链接，结构清晰，可以通过广播的方式发送消息到各个Client。

会话模式：这种模式一般称之为点对点的星型结构，Client与Server之间是直接的Peer2Peer的连接，少了DBus Daemon的中转，因此性能较好。

注意：在使用DBus通信前，需要保证系统安装了GIO_LIBS库，如果Linux环境还没有安装GIO_LIBS，请使用以下命令进行安装：  
ubuntu 16.04：  
Note: ubuntu 下安装依赖比较多
```
sudo apt-get install libglib-2.0-dev
```
fedora 25:
```
sudo dnf install libgnomeui-devel
```
##xml 文件使用

# Bluez dbus api 接口
## Bluez 版本
BLUEZ 5.44
## Bluez tool 使用
### 启动 bluetoothctl
```
omni@omni-PC:~$ bluetoothctl 
```
```
[NEW] Controller 70:B3:D5:4B:9C:19 omni-PC [default]
[NEW] Device E4:A4:71:55:67:74 CHN28150060
[NEW] Device 00:9E:C8:68:27:52 客厅的小米盒子
[NEW] Device B0:91:22:1A:EF:86 CC2640 R2
[NEW] Device 4F:B7:C4:48:17:9A 4F-B7-C4-48-17-9A
[NEW] Device 77:6B:36:4E:FE:29 77-6B-36-4E-FE-29
[NEW] Device 24:71:89:1A:9A:05 CC2650 RC
```
### 连接 设备
```
connect [device mac]
```
```
[bluetooth]# connect 24:71:89:1A:9A:05
Attempting to connect to 24:71:89:1A:9A:05
[CHG] Device 24:71:89:1A:9A:05 Connected: yes
[CHG] Device 24:71:89:1A:9A:05 Connected: no
Connection successful
[CC2650 RC]# 
```
### 查看设备信息
```
info [device mac]
```
```
[CC2650 RC]# info 24:71:89:1A:9A:05
Device 24:71:89:1A:9A:05
	Name: CC2650 RC
	Alias: CC2650 RC
	Appearance: 0x03c0
	Paired: yes
	Trusted: yes
	Blocked: no
	Connected: no
	LegacyPairing: no
	UUID: Generic Access Profile    (00001800-0000-1000-8000-00805f9b34fb)
	UUID: Generic Attribute Profile (00001801-0000-1000-8000-00805f9b34fb)
	UUID: Device Information        (0000180a-0000-1000-8000-00805f9b34fb)
	UUID: Battery Service           (0000180f-0000-1000-8000-00805f9b34fb)
	UUID: Human Interface Device    (00001812-0000-1000-8000-00805f9b34fb)
	UUID: Scan Parameters           (00001813-0000-1000-8000-00805f9b34fb)
	UUID: Vendor specific           (f000b000-0451-4000-b000-000000000000)
	Modalias: bluetooth:v000Dp0000d0110
	RSSI: -59
```
### 查看 attributes
```
list-attributes 24:71:89:1A:9A:05
```
## Bluez5 通过D-Bus API 控制蓝牙 
BlueZ5通过D-Bus提供API给第三方应用，所有的BlueZ API可以在BlueZ5源码的doc/目录下查到，这些API可以通过glib2.0中的GDBus来访问，下面的网址提供了所有可用的GDBus API： 
<https://developer.gnome.org/gio/stable/index.html>  
常用的Bluez5 API有adapter，agent，device以及profile，下面将具体介绍如何通过GDBus来访问这些常用BlueZ API： 
### 1.监控目标D-Bus 
GDBus中的g_bus_watch_name API可以很容易的实现对目标D-Bus的监控并获得GDBusConnection 对象：



