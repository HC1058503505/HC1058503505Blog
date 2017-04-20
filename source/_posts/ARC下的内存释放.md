---
title: ARC下的内存释放
date: 2017-04-20 10:31:34
tags:
---
ARC下的内存释放，避免内存泄漏。<!--more-->
# free()
```objc
-(void)objectConvertToDic{    
    unsigned int count = 0;
    Ivar *varList = class_copyIvarList([self.statisticsM class], &count);
    NSArray*keysArray = [NSArrayarray];

    if (kIsKindOfClass(self.statisticsM, MonthlyUserStatisticsModel)) {   
        // 用户数        
        self.title = @"用户数";        
        keysArray = self.userStatisticsKeys;          
  } else if (kIsKindOfClass(self.statisticsM, MonthlyTestStatisticsModel)) {  
       // 测试量        
       self.title = @"测试量";        
       keysArray = self.testStatisticsKeys;           
  } else if (kIsKindOfClass(self.statisticsM, MonthlyCellStatisticsModel)) {   
      // 测试小区        
      self.title = @"测试小区";        
      keysArray = self.cellStatisticsKeys;          
  } else if (kIsKindOfClass(self.statisticsM, MonthlyProblemStatisticsModel)) {   
      // 发现问题        
      self.title = @"发现问题";        
      keysArray = self.problemStatisticsKeys;    
  }       
  // 运行时获取模型属性与值,组成字典数组    
  for (int i = 0; i < count; i++) {       
        Ivar var = varList[i];        
        const char *varName = ivar_getName(var);       
        NSString *varStr = [NSString stringWithUTF8String:varName];               
        id value = [self.statisticsM valueForKeyPath:varStr];               
        value = (value == nil) ? @"-" : value;               

        NSMutableDictionary *tempDic = [NSMutableDictionary dictionary];       
        [tempDic setObject:value forKey:keysArray[i]];       
        [self.dataSourceArray addObject:tempDic];   
   }       
   free(varList);
}
```

# delete 与 delete[] 释放内存
  *  delete    释放new分配的单个对象指针指向的内存
```c
data_collections *dataCollection = &data_collections::getInstance();              
event_model *eventModel = new event_model();
eventModel->time = [[NSDate date] timeIntervalSince1970] * 1000;
eventModel->eventId = -1;
delete eventModel;
```
  * delete[ ] 释放new分配的对象数组指针指向的内存
```c
int *a = new int[10];
delete a;       //方式1,只会释放a指向数组的第一个元素
delete [] a;     //方式2,会逐一释放数组里的元素
```

# autoreleasePool
  1. 对于每一个Runloop， 系统会隐式创建一个Autorelease pool，这样所有的release pool会构成一个象CallStack一样的一个栈式结构，在每一个Runloop结束时，当前栈顶的     Autorelease pool会被销毁，这样这个pool里的每个Object会被release.
  2. 那什么是一个Runloop呢？ 一个UI事件，Timer call， delegate call， 都会是一个新的Runloop。Autorelease是保证一个method安全的，对于method中的函数调用也适用.
  3. 很多C/C++转过来的程序员会说，这个auto release有什么好，象C/C++那样，自己申请，自己释放，完全可控不好么， 这个auto relase 完全不可控，你都不知到它什么时候会被真正的release。我的理解它有一个作用就是可以做到每个函数对自己申请的对象负责，自己申请，自己释放，该函数的调用者不需要关心它内部申请对象的管理。 在下面这个例子中，Func1的调用者不需要再去关心obj的释放。

```objc
NSMutableArray *tempArray = [NSMutableArray array];
for ( int i = 0; i < 100000; i++) {
    @autoreleasepool {
        NSNumber *num = [NSNumber numberWithInt:i]; // 1
        NSString *str = [NSString stringWithFormat:@"%d ", i];  //2
        NSString *str1 = [NSString stringWithFormat:@"%@", num]; // 3
        NSString *str2 = [NSString stringWithFormat:@"%@%@", num, str]; //4

        Person *p = [[Person alloc] init];      
        p.name = @"HC";
        p.age = 25;
        [tempArray addObject:p];
    }
}
/// 没进行一次循环，常见许多临时变量，如果不释放的话内存会瞬间飙升，但是有了autoreleasepool会自动进行释放，避免内存飙升.
/// 注意autoreleasepool必须在for循环内部，如果在外部则和不适用autoreleasepool没什么区别.
```