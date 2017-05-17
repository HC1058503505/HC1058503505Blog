---
title: iOS蓝牙开发(一)
date: 2017-04-24 16:26:20
tags: iOS蓝牙开发
categories: iOS开发
---
蓝牙4.0的低能耗，使得蓝牙的应用越来越广泛，穿戴设备、智能家居等都有所涉及。本文主要介绍蓝牙开发相关基础知识，内容主要来自于[Apple官方开发文档][1]。<!--more-->
[1]:https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/CoreBluetoothOverview/CoreBluetoothOverview.html#//apple_ref/doc/uid/TP40013257-CH2-SW1

### 中心模式和外设模式
> CoreBluetooth框架用于蓝牙开发，其核心其实是centeral和peripheral，可理解为中心模式和外设模式。

中心模式，就是以你的设备为中心，连接其他的外围设备；外设模式，是使你的设备被其他中心设备连接。蓝牙应用是基于传统的`客户端-服务器`结构，外围设备有其他设备需要的数据，中心设备使用外围设备提供的数据执行一系列任务。
![CBDevices-image](/images/CBDevices1.png)

### 中心设备搜索和连接正在广告的外围设备
外围设备对外广播广告包形式的数据，广告包含了外设的名字，功能服务。另一方面，中心设备可以扫描和监听正在广播广告的外设。
![AdvertisingAndDiscovery-image](/images/AdvertisingAndDiscovery.png)

### 外围设备的数据结构
外围设备包含一个或多个服务(services)以及提供了连接信号的强度。一个服务是一些数据的集合，是设备完成一个特定行为的功能。例如，一个心率计的服务是从心率传感器读取数据。服务由特征（characteristics）组成。一个特征提供外围设备服务的更详细信息。例如，心率服务只是说明它包含一个描述心率传感器所在的身体部位的特征和一个传输心率测量数据的特征。
![CBPeripheralData-image](/images/CBPeripheralData.png)

### 中心设备检测外围设备并与其进行数据交互
一个中心设备与一个外围设备连接成功后，便可以搜索外围设备提供的所有服务和特征。（广告的数据可能只含有一部分可用服务）。一个中心设备也可以通过读和写服务的特征来与外围设备交互。例如，你的app可以从数码温度调节器请求当前房间的温度，或者可以提供一个值来设置房间温度。

### 中心设备、外围设备和外围设备数据的表现形式
主要的蓝牙参与者和相关数据对应Core Bluetooth framework中的哪些对象？

- 中心设备
	1.本地中心设备和远程外围设备
	在中心设备这边，一个本地中心设备用一个CBCentralManager对象来代表，这些对象用来管理发现和连接远程设备（用CBPeripheral对象来代表），包括搜索、发现、连接正在广告的外围设备。如下图，显示本地中心设备和远程外围设备在Core Bluetooth framework的表现形式。
	![CBObjects_CentralSide-image](/images/CBObjects_CentralSide.png)
	2.远程外围设备数据用CBService 和 CBCharacteristic来表示
	当你与外围设备（ CBPeripheral对象）进行数据交互，你需要处理它的服务（services）和特征（characteristics），在Core Bluetooth framework中，外围设备的服务用CBService对象来表示。同样的，外围设备的特征用 CBCharacteristic对象来表示
	![TreeOfServicesAndCharacteristics_Remote-image](/images/TreeOfServicesAndCharacteristics_Remote.png)

- 外围设备对应的对象
> 从OS X10.9 和iOS6开始，Mac和iOS设备可以作为蓝牙4.0的外围设备，为其它设备提供数据，包括其它的Mac、iPhone和iPad设备。当你将你的设备设置为外围设备角色，你需要实现外围设备的一些处理。

	1.本地外围设备和外部中心设备
	在外围设备这一边，本地外围设备用 CBPeripheralManager对象来表示，这些对象用来管理外围设备数据库中的服务和特征，以及给外部的中心设备（CBCentral对象）广播这些服务，外围设备管理对象同时可以回应远程设备的读和写请求，下图显示本地外围设备和远程中心设备在 Core Bluetooth framework的表示
	![CBObjects_PeripheralSide-image](/images/CBObjects_PeripheralSide.png)
	2.本地外围设备的数据使用CBMutableService 和CBMutableCharacteristic对象来表示
	当你设置本地外围设备（ CBPeripheralManager对象）并且进行数据交互，你需要处理它的服务和特征。在Core Bluetooth framework中，本地外围设备的服务用CBMutableService对象来表示。同样的，一个服务的特征用CBMutableCharacteristic对象来表示，图1-7便是本地外围设备的服务和特征的基本结构
	![TreeOfServicesAndCharacteristics_Local-image](/images/TreeOfServicesAndCharacteristics_Local.png)