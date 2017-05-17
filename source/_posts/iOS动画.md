---
title: iOS动画
date: 2017-04-28 15:56:19
tags: iOS动画
categories: iOS开发
---
本章主要介绍iOS动画的基本使用，是通过一个一个实例来介绍iOS动画的基本使用。<!--more-->

## UIView Animation

```objc
// 1.添加Label
UILabel *label = [[UILabel alloc] initWithFrame:CGRectMake(100, 300, 100, 100)];
label.numberOfLines = 0;
label.text = @"UIViewAnimation";
label.backgroundColor = [UIColor orangeColor];
label.userInteractionEnabled = true;
[self.view addSubview:label];

// 2.添加手势
UITapGestureRecognizer *gesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapAction:)];
[label addGestureRecognizer:gesture];

// 3.执行动画
- (void)tapAction:(UIGestureRecognizer *)gesture {
    CGPoint endPoint = CGPointMake(gesture.view.layer.position.x + 100, gesture.view.layer.position.y);

    [UIView animateWithDuration:1.0 animations:^{
         gesture.view.layer.position = endPoint;
    }];
//        [UIView animateWithDuration:1.0 animations:^{
//        } completion:^(BOOL finished) {
//        }];

//        // 延迟动画
//        [UIView animateWithDuration:1.0 delay:0.5 options:UIViewAnimationOptionCurveEaseInOut animations:^{
//        } completion:^(BOOL finished) {
//        }];

}
```

## CoreAnimation

![coreAnimation-image](/images/coreAnimation.png)

1. CAAnimation：核心动画的基础类，不能直接使用，负责动画运行时间、速度的控制，本身实现了CAMediaTiming协议。
2. CAPropertyAnimation：属性动画的基类（通过属性进行动画设置，注意是可动画属性），不能直接使用。
3. CAAnimationGroup：动画组，动画组是一种组合模式设计，可以通过动画组来进行所有动画行为的统一控制，组中所有动画效果可以并发执行。
4. CATransition：转场动画，主要通过滤镜进行动画效果设置。
5. CABasicAnimation：基础动画，通过属性修改进行动画参数控制，只有初始状态和结束状态。
6. CAKeyframeAnimation：关键帧动画，同样是通过属性进行动画参数控制，但是同基础动画不同的是它可以有多个状态控制。

### CABaseAnimation
```objc
// 1.添加view
UIView *redView = [[UIView alloc] initWithFrame:CGRectMake(100, 100, 100, 100)];
self.redView = redView;
redView.backgroundColor = [UIColor redColor];
[self.view addSubview:redView];
// 2.添加手势
UITapGestureRecognizer *gesture = [[UITapGestureRecognizer alloc] initWithTarget:self action:@selector(tapAction:)];
[redView addGestureRecognizer:gesture];

- (void)tapAction:(UIGestureRecognizer *)gesture {
    CGPoint endPoint = CGPointMake(gesture.view.layer.position.x + 100, gesture.view.layer.position.y);
    CGPoint startPoint = gesture.view.layer.position;
    // 创建动画实例
    CABasicAnimation *baseAnimation = [CABasicAnimation animationWithKeyPath:@"position"];
    // 设置动画代理
    baseAnimation.delegate = self;
    // 动画起始位置
    baseAnimation.fromValue = [NSValue valueWIthCGPoint:startPoint];
    // 动画结束位置
    baseAnimation.toValue = [NSValue valueWithCGPoint:endPoint];
    // 动画执行时长
    // baseAnimation.timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)//加速运动
    // baseAnimation.timingFunction = CAMediaTimingFunction(controlPoints: 0.5, 0, 0.9, 0.7)//自定义加速的曲线参数
    baseAnimation.duration = 1.0;

    // 设置动画执行测试 baseAnimation.repeatCount = 1;
    // 在动画中存储一些必要数据
    [baseAnimation setValue:[NSValue valueWithCGPoint:endPoint] forKey:@"redView"];

    // 避免动画执行后回复位
    baseAnimation.removedOnCompletion = false;
    baseAnimation.fillMode = kCAFillModeForwards;
    // 图层添加动画
    gesture.view.layer addAnimation:baseAnimation forKey:nil]; 
}
```