---
title: RunLoop
date: 2017-07-06 12:53:04
tags: RunLoop
categories: iOS开发
---
> 这是一次电话面试的问题，当时很模糊，感觉作为一个iOS开发者还是要做一个了解，下面借助度娘总结一下，用的着的点赞。<!-- more -->

# 什么是RunnLoop
RunLoop是一个"死"循环，应为有它的存在，保证了App能够持续运行。在一个iOS程序启动后，在UIApplicationMain函数内部启动一个跟主线程相关的RunnLoop对象，而UIApplicationMain函数一直运行从而保证程序一直运行。
![run loop](/images/RunLoop.png)
通过所有的“消息”都被添加到了RunLoop中去，而在这里这些消息又分为“input source”和“Timer source” 并在循环中检查是不是有事件需要发生，如果需要那么就调用相应的函数处理。由此形成了运行->检测->休眠 ->运行 的循环状态。

# 主要作用
* 使程序一直运行并接受用户输入
* 决定程序在何时处理一些事件
* 条件解耦，消息队列
* 节省CPU资源，有事时处理，没事是休眠

# RunLoop与线程之间的关系
RunLoop是用来管理线程的。每一个线程都有一个RunLoop对象。可以通过具体的方法去获得。但是需要注意：虽然每一个线程都可以获取RUnLoop对象，但是并不是每一个线程中都有实例对象，我们可以这样理解：如果我们不获取RunLoop，这个RunLoop就不存在，我们获取时，如果不存在，就会去创建。在主线程中，这个主RunLoop是默认创建并运行激活的。

# iOS开辟线程占用的空间情况
线程创建的成本：
```yml
kernel data structures    约1KB
Stack space               512KB(secondary threads) 
                          1MB(iOS main thread)
Creation time             约90 microseconds
```
当编写线程代码时另外一个需要考虑的成本是生产成本。线程的滥用会占用大量的内存，使用需谨慎。

# Mode
Mode中有三个非常重要的组成部分，Timer(定时器)、Source(事件源)以及Observor(观察者)。一个RunLoop包含若干个Mode，每个Mode又包含若干个Source/Timer/Observer。首先要指出的是一个runloop启动时必须指定一个Mode,并且这个Mode被称为currentMode。如果要切换Mode,只能退出runloop重新进入。这样做主要是为了分隔开不同组的 Source/Timer/Observer，让其互不影响。随后我们会分别介绍每一类的具体作用与应用场景。
系统默认注册的Mode有五种
```objc

kCFRunloopDefaultMode    // App默认Mode   通常主线程是在这个mode下运行

UITrackingRunloopMode    // 界面跟踪Mode  用于scrollView追踪触摸  界面滑动时不受其他Mode影响

UIinitializationRunloopMode    //在app一启动进入的第一个Mode,启动完成后就不再使用

GSEventRecieveRunloopMode   //苹果使用绘图相关

NSRunLoopCommonModes 　　//占位模式
```

# 实际应用
## 只在NSRUnloopDefaultModes 下显示图片
```objc
//在cell里面把设置图片的事情在NSDefaultRunloopMode里面去做。
//当主线程的tableview不再滑动的时候就会去设置图片
UIImage *dowloadImage = ...;
[self.iconImageView performSelector:@selector(setImage:) withObject:dowloadImage afterDelay:0 inModes:@[NSDefaultRunloopMode]];
```
## NSTimer NSRunLoopCommonModes
```objc
self.timer = [NSTimer scheduledTimerWithTimeInterval:0.0625
                                               target:self
                                             selector:@selector(progressChange)
                                             userInfo:nil
                                              repeats:YES];

 [[NSRunLoop currentRunLoop] addTimer:self.timer forMode:NSRunLoopCommonModes];
```
这样声明的NSTimer可以解决在滑动scrollView时NSTimer不工作的问题。forMode:NSRunLoopCommonModes的意思为，定时器可以运行在标记为common modes模式下。具体包括两种: kCFRunloopDefaultMode和UITrackingRunloopMode。当你开始滑动UIScrollView时，RunLoop的mode状态变化如下：
```objc
NSDefaultRunLoopMode -> UITrackingRunLoopMode -> NSDefaultRunLoopMode
```
## RunLoop与GCD定时器
GCD定时器的优势:不受RunLoop的运行模式的影响
```swift
class ViewController: NSViewController {
var timerSource:DispatchSourceTimer!
var isStop:Bool = false
override func viewDidLoad() {
    super.viewDidLoad()
    // DispatchSourceTimer
    let timer = DispatchSource.makeTimerSource(flags: [], queue: DispatchQueue.main)
    timerSource = timer // 注意:需要保持拥有，不然会立即释放掉
    timerSource.scheduleRepeating(deadline: .now(), interval: .microseconds(40))
    
    timerSource.setEventHandler {
        print("Hello")
    }
    timerSource.resume()
    // Timer
    // time会被添加到RunnLoop当中，不需要保持拥有
    let time = Timer(fire: Date.distantPast, interval: 0.5, repeats: true) { (time) in
        print("Timer")
    }
    
    RunLoop.current.add(time, forMode: .defaultRunLoopMode)
    
}

@IBAction func stop(_ sender: NSButton) {
    
    if isStop {
        timerSource.resume()
    } else {
        timerSource.suspend()
    }
    
    isStop = !isStop
}
```
## AFNetworking 常驻线程
通常执行完方法后线程就销毁了，那么现在有这样的需求，需要一条子线程一直存在，等待处理任务，与主线程之间互不干扰  (可以类比主线程存在原理，即添加消息循环Runloop)
```objc
+ (void)networkRequestThreadEntryPoint:(id)_unuserd object {
    @autoreleasepool {
        [[NSThread currentThread] setName:@"AFNetworking"];

        //为了不让runloop run起来没事干导致消失
        //所以给runloop加了一个NSMachPort，给它一个mode去监听
        //实际上port什么也没干，就是让runloop一直在等，目的就是让runloop一直活着
        //这是一个创建常驻服务线程的好方法
        NSRunloop *runloop = [NSRunLoop currentRunLoop];
        [runloop addPort:[NSMachPort port] forMode:NSDefaultRunLoopMode];
        [runloop run];
    }
}

+ (NSThread *)networkRequestThread {
    static NSThread *_networkReuqestThread = nil;
    static dispatch_once_t oncePredicate;
    dispatch_once(&oncePredicate, ^{
        _networkRequestThread = 
        [[NSThread alloc] initWithTarget:self selector:@selector(networkRequestThreadEntryPoint:) object:nil];
        [_networkRequestThread start];
    });
    return _networkRequestThread;
}
```
Ps:Runloop运行首先判断Mode是否为空，如果为空则退出循环，还可以通过removePort来移除端口。本例用添加port来实现，其他方法请读者自己多尝试。

## 关于自动释放池 

关于自动释放池，子线程开启runloop时要开启针对当前线程的autoreleasepool，在每次NSRunloop休眠前清理自动释放池。

参考文章:
1.[浅谈NSRunloop工作原理和相关应用](http://www.cnblogs.com/ruihaha/p/5813819.html)
2.[iOS NSRunloop详解](http://www.jianshu.com/p/296f182c8faa)
