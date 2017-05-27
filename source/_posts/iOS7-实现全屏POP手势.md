---
title: iOS7+ 实现全屏POP手势
date: 2017-05-27 15:59:33
tags: iOS开发技巧
categories: iOS开发
---
> 我们知道, iOS7的导航控制器有一个自带的pop手势(interactivePopGestureRecognizer), 该手势允许我们从屏幕左侧向右滑动来pop掉当前的栈顶控制器. 事实上, 我们看到国内许多app就是采用了系统自带的这个recognizer, 并没有定制切换效果. 而也有很多app(如最新版的网易新闻)在维持系统pop效果的前提下, 把手势响应区域扩展到全屏!这里对这一实现过程进行总结。<!--more -->

## 问题发掘

导航栏的pop手势只能够响应左侧有限的区域内，内部实现时肯定是做了限制，这种限制应该也是对手势属性值或方法的设置来实现，那么我们需要看一下该手势的具体实现，当然是使用runtime。
```swift
let gesture = navigationController?.interactivePopGestureRecognizer
print("gesture的类名:\(NSStringFromClass((gesture?.classForCoder)!))")
print("gesture的父类名:\(NSStringFromClass((gesture?.superclass)!))")

var ivarCount:UInt32 = 0
let ivarList = class_copyIvarList(gesture?.classForCoder, &ivarCount)
for i in 0 ..< ivarCount {
    let ivar = ivarList![Int(i)]
    let name = ivar_getName(ivar)
    print("class_copyIvarList---\(String(cString: name!, encoding: .utf8) ?? "")")
}
free(ivarList)


var proprtyCount:UInt32 = 0
let propertyList = class_copyPropertyList(gesture?.classForCoder, &proprtyCount)
for i in 0 ..< proprtyCount {
    let property = propertyList![Int(i)]
    print("class_copyPropertyList---\(String(cString: property_getName(property), encoding: .utf8) ?? "")")
}
free(propertyList)
```
结果如下：
```objc
gesture的类名:UIScreenEdgePanGestureRecognizer
gesture的父类名:UIPanGestureRecognizer
class_copyIvarList---_recognizer
class_copyPropertyList---edges
class_copyPropertyList---hash
class_copyPropertyList---superclass
class_copyPropertyList---description
class_copyPropertyList---debugDescription
```
可见是`UIScreenEdgePanGestureRecognizer`私有类的edges属性决定了pop手势的作用范围。我们怎么利用这一特性呢？
1. 设置其edges达到全屏pop
2. 想想我们添加手势处理事件时需要`target-action`,我们可以利用该手势的`targe-action`

这里第一种方法以后尝试，我们现在使用第二种方法来实现。


## 寻找target-action
![interactivePopGestureRecognizer](/images/interactivePopGestureRecognizer.png)
由上图可知，interactivePopGestureRecognizer的类型为UIGestureRecognizer,所以
```swift
var ivarGesture:UInt32 = 0
let ivarGestureList = class_copyIvarList(UIGestureRecognizer.self, &ivarGesture)
for i in 0 ..< ivarGesture {
    let ivar = ivarGestureList![Int(i)]
    let name = ivar_getName(ivar)
    let type = ivar_getTypeEncoding(ivar)
    print("ivarGestureName---\(String(cString: name!, encoding: .utf8) ?? "")   ivarGestureType:\(String(cString: type!, encoding: .utf8) ?? "")")
}

free(ivarGestureList)
```
结果如下：
```swift
ivarGestureName---_gestureFlags   ivarGestureType:{?="delegateShouldBegin"b1"delegateCanPrevent"b1"delegateCanBePrevented"b1"delegateShouldRecognizeSimultaneously"b1"delegateShouldReceiveTouch"b1"delegateShouldReceivePress"b1"delegateShouldRequireFailure"b1"delegateShouldBeRequiredToFail"b1"delegateFailed"b1"privateDelegateShouldBegin"b1"privateDelegateCanPrevent"b1"privateDelegateCanBePrevented"b1"privateDelegateShouldRecognizeSimultaneously"b1"privateDelegateShouldReceiveTouch"b1"privateDelegateShouldReceivePress"b1"privateDelegateShouldRequireFailure"b1"privateDelegateShouldBeRequiredToFail"b1"subclassShouldRequireFailure"b1"subclassShouldBeRequiredToFail"b1"hasSubclassDynamicFailureRequirements"b1"hasDelegateDynamicFailureRequirements"b1"subclassTouchesEstimatedPropertiesUpdated"b1"queriedFailureRequirements"b1"cancelsTouchesInView"b1"delaysTouchesBegan"b1"delaysTouchesEnded"b1"disabled"b1"dirty"b1"delivered"b1"deliveredEndedOrCancelled"b1"continuous"b1"requiresDelayedBegan"b1"willBeginAfterSatisfyingFailureRequirements"b1"requiresSystemGesturesToFail"b1"acceptsFailureRequirements"b1"requiresExclusiveTouchType"b1"initialTouchTypeIsValid"b1"forceRequirementSatisfied"b1}
ivarGestureName---_targets   ivarGestureType:@"NSMutableArray"
ivarGestureName---_delayedTouches   ivarGestureType:@"NSMutableArray"
ivarGestureName---_delayedPresses   ivarGestureType:@"NSMutableArray"
ivarGestureName---_view   ivarGestureType:@"UIView"
ivarGestureName---_lastTouchTimestamp   ivarGestureType:d
ivarGestureName---_state   ivarGestureType:q
ivarGestureName---_allowedTouchTypes   ivarGestureType:q
ivarGestureName---_initialTouchType   ivarGestureType:q
ivarGestureName---_internalActiveTouches   ivarGestureType:@"NSMutableSet"
ivarGestureName---_forceClassifier   ivarGestureType:@"_UIForceLevelClassifier"
ivarGestureName---_requiredPreviewForceState   ivarGestureType:q
ivarGestureName---_touchForceObservable   ivarGestureType:@"_UITouchForceObservable"
ivarGestureName---_touchForceObservableAndClassifierObservation   ivarGestureType:@"NSObservation"
ivarGestureName---_forceTargets   ivarGestureType:@"NSMutableArray"
ivarGestureName---_forcePressCount   ivarGestureType:Q
ivarGestureName---_beganObservable   ivarGestureType:@"NSObservationSource"
ivarGestureName---_failureRequirements   ivarGestureType:@"NSMutableSet"
ivarGestureName---_failureDependents   ivarGestureType:@"NSMutableSet"
ivarGestureName---_delegate   ivarGestureType:@"<UIGestureRecognizerDelegate>"
ivarGestureName---_allowedPressTypes   ivarGestureType:@"NSArray"
ivarGestureName---_gestureEnvironment   ivarGestureType:@"UIGestureEnvironment"
```
我猜`_targets`就是我们要找的，感觉离成功不远了
```swift
// 获取targets
let targets = navigationController?.interactivePopGestureRecognizer?.value(forKeyPath: "_targets") as! Array<Any>
for target in targets {
	print(target)
}
```
结果如下：
```swift
(action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 0x7fe95d514850>)
```
成功，action，target全部拿到，下面就是添加我们自己的手势然后设置action，target。
```swift
// 全屏pop手势
let targets = navigationController?.interactivePopGestureRecognizer?.value(forKeyPath: "_targets") as! Array<Any>

let obj = targets.last! // (action=handleNavigationTransition:, target=<_UINavigationInteractiveTransition 0x7fe95d514850>)
let selector = Selector(("handleNavigationTransition:"))
let target = (obj as AnyObject).value(forKeyPath: "target") // navigationController?.interactivePopGestureRecognizer?.delegate
let pan = UIPanGestureRecognizer(target: target, action: selector)

view.addGestureRecognizer(pan)

```
最后经尝试发现interactivePopGestureRecognizer的代理就是pop手势的target。

参考链接：
[KVC实现全屏pop手势(iOS7+)](http://www.jianshu.com/p/72651cdbd914)
[喵神-iOS7中的ViewController切换](https://onevcat.com/2013/10/vc-transition-in-ios7/)