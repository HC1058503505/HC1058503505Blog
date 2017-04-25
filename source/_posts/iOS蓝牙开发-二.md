---
title: iOS蓝牙开发(二)
date: 2017-04-25 11:19:22
categories: iOS蓝牙开发
---
iOS蓝牙中心模式连接外设，数据接收及发送。<!--more-->

## 连接外设流程
```yml
建立Central Manager实例进行蓝牙管理
搜索,连接外设
获取外设的服务,特征
向外设特征值发送读取和写入请求
订阅一特征值去监控其变化
```
## 实现步骤

### 创建蓝牙管理者
> CBCentralManager是Core Bluetooth的一个对象，代表一个本地中心设备，在使用蓝牙前，需要创建CBCenterManager实例。

```
centralM = [[CBCentralManager alloc] initWithDelegate:self queue:nil options:nil];
```
创建蓝牙管理者时,`self`会被设置为接收中心设备所有事件的代理者，参数`queue`设置为nil后，`centralM`会通过主线程来处理事件。控制器需要实现`CBCentralManagerDelegate`、`CBPeripheralDelegate`这两个代理。当创建了一个蓝牙管理者，会回调代理的`centralManagerDidUpdateState:`方法，你必须实现这个代理方法来确定中心设备是否支持BLE以及是否可用。
```objc
- (void)centralManagerDidUpdateState:(CBCentralManager *)central {
    switch (central.state) {
        case CBManagerStateUnknown: // 蓝牙状态未知 State unknown, update imminent.
            NSLog(@">>>CBManagerStateUnknown");
            break;
        case CBManagerStateResetting: // 蓝牙重置 The connection with the system service was momentarily lost, update imminent.
            NSLog(@">>>CBManagerStateResetting");
            break;
        case CBManagerStateUnsupported: // 不支持 The platform doesn't support the Bluetooth Low Energy Central/Client role.
            NSLog(@">>>CBManagerStateUnsupported");
            break;
        case CBManagerStateUnauthorized: // 未授权 The application is not authorized to use the Bluetooth Low Energy role.
            NSLog(@">>>CBManagerStateUnauthorized");
            break;
        case CBManagerStatePoweredOff: // 蓝牙未开启 Bluetooth is currently powered off.
            NSLog(@">>>CBManagerStatePoweredOff");
            break;
        default: // 蓝牙开启状态，可使用 Bluetooth is currently powered on and available to use.
            NSLog(@">>>CBManagerStatePoweredOn");
            // 开始扫描外设
            // 如果你指定第一个参数为nil,centralM会返回发现的所有设备.
            // 实际开发中，你需要指定一个`CBUUID`对象数组,数组中每一个对象对应一个唯一的外设的服务
            [centralM scanForPeripheralsWithServices:nil options:nil];
            break;
    }
}
```
### 搜索连接外设
> 调用了`scanForPeripheralsWithServices:options:`方法来搜索可连接的外围设备后,centralM在每次搜索到一个外围设备都会回调其代理的`centralManager:didDiscoverPeripheral:advertisementData:RSSI:` 方法。任何被搜索到的外围设备都以``CBPeripheral``类的方式返回。像下面的代码，你可以实现这个代理方法来罗列出所有被搜索到的蓝牙设备

```objc
- (void)centralManager:(CBCentralManager *)central 
 didDiscoverPeripheral:(CBPeripheral *)peripheral 
     advertisementData:(NSDictionary<NSString *,id> *)advertisementData 
                  RSSI:(NSNumber *)RSSI {
  
    NSLog(@"%@",peripheral.name);
    if (peripheral.name.length > 0) {
    	// 找到的设备必须持有它，否则CBCentralManager中也不会保存peripheral，那么CBPeripheralDelegate中的方法也不会被调用！！
        [peripherals addObject:peripheral]; 
        // [manager stopScan]; // 根据需求可以停止搜索其他外设
        [centralM connectPeripheral:peripheral options:nil]; // 连接外设
    }
}

 //连接到Peripherals-成功
 - (void)centralManager:(CBCentralManager *)central didConnectPeripheral:(CBPeripheral *)peripheral{
     NSLog(@">>>连接到名称为（%@）的设备-成功",peripheral.name);
    // 设置外设代理
    [peripheral setDelegate:self];
    // 搜索外设的服务
    [peripheral discoverServices:nil];
 }

//连接到Peripherals-失败
-(void)centralManager:(CBCentralManager *)central didFailToConnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error{
    NSLog(@">>>连接到名称为（%@）的设备-失败,原因:%@",[peripheral name],[error localizedDescription]);
}

//Peripherals断开连接
- (void)centralManager:(CBCentralManager *)central didDisconnectPeripheral:(CBPeripheral *)peripheral error:(NSError *)error{
     NSLog(@">>>外设连接断开连接 %@: %@\n", [peripheral name], [error localizedDescription]);
}
```

### 扫描外设中的服务器和特征
> 设备连接成功后，就可以扫描设备的服务了，同样是通过委托形式，扫描到结果后会进入委托方法。但是这个委托已经不再是主设备的委托`CBCentralManagerDelegate`,而是外设的委托`CBPeripheralDelegate`,这个委托包含了主设备与外设交互的许多回叫方法，包括获取services，获取characteristics，获取characteristics的值，获取characteristics的Descriptor，和Descriptor的值，写数据，读rssi，用通知的方式订阅数据等等。

#### 获取外设的services
```objc
//扫描到Services
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverServices:(NSError *)error{
    //  NSLog(@">>>扫描到服务：%@",peripheral.services);
    if (error)
    {
        NSLog(@">>>Discovered services for %@ with error: %@", peripheral.name, [error localizedDescription]);
        return;
    }

    for (CBService *service in peripheral.services) {
        NSLog(@"%@",service.UUID);
        // 扫描每个service的Characteristics，扫描到后会进入方法： 
        // -(void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error
        [peripheral discoverCharacteristics:nil forService:service];
    }

}

```
#### 获取服务的特征
```objc
// 搜索服务的特征
- (void)peripheral:(CBPeripheral *)peripheral didDiscoverCharacteristicsForService:(CBService *)service error:(NSError *)error {
    if (error)
    {
        NSLog(@"error Discovered characteristics for %@ with error: %@", service.UUID, [error localizedDescription]);
        return;
    }
    
    for (CBCharacteristic *characteristic in service.characteristics)
    {
        NSLog(@"service:%@ 的 Characteristic: %@",service.UUID,characteristic.UUID);
        
         // 获取Characteristic的值，当你试图去读一个特征对应的值，外围设备会回调它的代理方法：
         // -(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
        [peripheral readValueForCharacteristic:characteristic];
        
        
        // 搜索Characteristic的Descriptors，读到数据会进入方法：
        // -(void)peripheral:(CBPeripheral *)peripheral didDiscoverDescriptorsForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error
        [peripheral discoverDescriptorsForCharacteristic:characteristic];
    }
    
}

// 获取的charateristic的值
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
    //打印出characteristic的UUID和值
    //!注意，value的类型是NSData，具体开发时，会根据外设协议制定的方式去解析数据
    NSLog(@"characteristic uuid:%@  value:%@",characteristic.UUID,characteristic.value);
    
}

//搜索到Characteristic的Descriptors
-(void)peripheral:(CBPeripheral *)peripheral didDiscoverDescriptorsForCharacteristic:(CBCharacteristic *)characteristic error:(NSError *)error{
    
    //打印出Characteristic和他的Descriptors
    NSLog(@"characteristic uuid:%@",characteristic.UUID);
    for (CBDescriptor *d in characteristic.descriptors) {
        NSLog(@"Descriptor uuid:%@",d.UUID);
    }
    
}

//获取到Descriptors的值
-(void)peripheral:(CBPeripheral *)peripheral didUpdateValueForDescriptor:(CBDescriptor *)descriptor error:(NSError *)error{
    //打印出DescriptorsUUID 和value
    //这个descriptor都是对于characteristic的描述，一般都是字符串，所以这里我们转换成字符串去解析
    NSLog(@"characteristic uuid:%@  value:%@",[NSString stringWithFormat:@"%@",descriptor.UUID],descriptor.value);
}
```

### 把数据写到Characteristic中
```objc
-(void)writeCharacteristic:(CBPeripheral *)peripheral
            characteristic:(CBCharacteristic *)characteristic
                     value:(NSData *)value{
    
    //打印出 characteristic 的权限，可以看到有很多种，这是一个NS_OPTIONS，就是可以同时用于好几个值，常见的有read，write，notify，indicate，知知道这几个基本就够用了，前连个是读写权限，后两个都是通知，两种不同的通知方式。
    /*
     typedef NS_OPTIONS(NSUInteger, CBCharacteristicProperties) {
     CBCharacteristicPropertyBroadcast												= 0x01,
     CBCharacteristicPropertyRead													= 0x02,
     CBCharacteristicPropertyWriteWithoutResponse									= 0x04,
     CBCharacteristicPropertyWrite													= 0x08,
     CBCharacteristicPropertyNotify													= 0x10,
     CBCharacteristicPropertyIndicate												= 0x20,
     CBCharacteristicPropertyAuthenticatedSignedWrites								= 0x40,
     CBCharacteristicPropertyExtendedProperties										= 0x80,
     CBCharacteristicPropertyNotifyEncryptionRequired NS_ENUM_AVAILABLE(NA, 6_0)		= 0x100,
     CBCharacteristicPropertyIndicateEncryptionRequired NS_ENUM_AVAILABLE(NA, 6_0)	= 0x200
     };
     
     */
    NSLog(@"%lu", (unsigned long)characteristic.properties);
    
    
    //只有 characteristic.properties 有write的权限才可以写
    if(characteristic.properties & CBCharacteristicPropertyWrite){
        /*
         最好一个type参数可以为CBCharacteristicWriteWithResponse或type:CBCharacteristicWriteWithResponse,区别是是否会有反馈
         */
        [peripheral writeValue:value forCharacteristic:characteristic type:CBCharacteristicWriteWithResponse];
    }else{
        NSLog(@"该字段不可写！");
    }
    
    
}

```
### 订阅Characteristic的通知
```objc
//设置通知
-(void)notifyCharacteristic:(CBPeripheral *)peripheral
             characteristic:(CBCharacteristic *)characteristic{
    //设置通知，数据通知会进入：didUpdateValueForCharacteristic方法
    [peripheral setNotifyValue:YES forCharacteristic:characteristic];
    
}

//取消通知
-(void)cancelNotifyCharacteristic:(CBPeripheral *)peripheral
                   characteristic:(CBCharacteristic *)characteristic{
    
    [peripheral setNotifyValue:NO forCharacteristic:characteristic];
}

```

### 停止扫描并断开连接
```objc
//停止扫描并断开连接
-(void)disconnectPeripheral:(CBCentralManager *)centralManager
                 peripheral:(CBPeripheral *)peripheral{
    //停止扫描
    [centralM stopScan];
    //断开连接
    [centralM cancelPeripheralConnection:peripheral];
}
```
---

参考文章:
[ImJackXu](http://blog.csdn.net/dolacmeng/article/details/46457487)
[liuyanwei](http://liuyanwei.jumppo.com/2015/08/14/ios-BLE-2.html)
[Core Bluetooth Programing Guide](https://developer.apple.com/library/content/documentation/NetworkingInternetWeb/Conceptual/CoreBluetooth_concepts/AboutCoreBluetooth/Introduction.html#//apple_ref/doc/uid/TP40013257-CH1-SW1)