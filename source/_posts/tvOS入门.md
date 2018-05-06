---
title: tvOS入门
date: 2018-05-05 22:38:23
tags: tvOS入门
categories: tvOS
---

> 初次接触到tvOS，和iOS开发有很大不同，主要体现在两者的交互模式。iOS以接受屏幕点击事件进行交互，tvOS通过遥控器聚焦的方式进行交互。这样的不同给iOS开发者转tvOS开发有些不适应，这里做下在开发中遇到的问题以及解决的办法。<!--more-->

## 剧情交代
进入CGTN这家公司，第一个任务就是搜索界面的键盘自定义，系统没有暴露属性来修改键盘样式，尝试使用KVC的方式来解决，但是较为困难，并且有使用私有API而被拒的风险，最终选择封装这样的一个键盘。封装键盘的过程中遇到一个核心问题——`聚焦`。默认情况下`UIButton`,`UICollection`,`UITableView`是可以自动被聚焦的，在聚焦变动的情况下，系统帮我们需在最近的控件为聚焦状态，当然也有可能找不到的情况。

![focusGuide-image](/images/focusGuide.png)


## UIFocusGuide
```objc
// 这时我们可以使用`UIFocusGuide`来做个连接，打通topRight与bottomLeft间的隔阂。

let focusGuide = UIFocusGuide()
view.addLayoutGuide(focusGuide)
        
// Anchor the top left of the focus guide.
focusGuide.leftAnchor.constraint(equalTo: topRightButton.leftAnchor).isActive = true
focusGuide.topAnchor.constraint(equalTo: bottomLeftButton.topAnchor).isActive = true
        
// Anchor the width and height of the focus guide.
focusGuide.widthAnchor.constraint(equalTo: topRightButton.widthAnchor).isActive = true
focusGuide.heightAnchor.constraint(equalTo: bottomLeftButton.heightAnchor).isActive = true

// MARK: UIFocusEnvironment
    
override func didUpdateFocus(in context: UIFocusUpdateContext, with coordinator: UIFocusAnimationCoordinator) {
    super.didUpdateFocus(in: context, with: coordinator)
        
    /*
        Update the focus guide's `preferredFocusedView` depending on which
        button has the focus.
    */
    guard let nextFocusedView = context.nextFocusedView else { return }

    switch nextFocusedView {
    	// topRIghtButton向下时focusGuide指向bottomLeftButton
        case topRightButton:
            focusGuide.preferredFocusEnvironments = [bottomLeftButton]
        
        // bottomLeftButton向上时focusGuide指向topRightButton
        case bottomLeftButton:
            focusGuide.preferredFocusEnvironments = [topRightButton]
            
        default:
            focusGuide.preferredFocusEnvironments = []
    }
}

```

## UICollectionView的聚焦问题
- 自定义聚焦位置
```objc
// MARK: collectview聚焦
func collectionView(_ collectionView: UICollectionView, didUpdateFocusIn context: UICollectionViewFocusUpdateContext, with coordinator: UIFocusAnimationCoordinator) {
    
    if let collectionViewFocused = self.isFocusedCollectionViewAction {
        collectionViewFocused(context.nextFocusedView! is CGTNCollectionViewCell)
    }
    
}

// MARK: 定义聚焦位置
func indexPathForPreferredFocusedView(in collectionView: UICollectionView) -> IndexPath? {
    return IndexPath(item: 1, section: 0)
}
```
- 记住聚焦位置
```objc
let collectionV = UICollectionView(frame:bounds, collectionViewLayout: flowLayout)
collectionV.isScrollEnabled = false
collectionV.delegate = self
collectionV.dataSource = self

/*
 * defaults to NO. 
 * If YES, when focusing on a collection view the 
 * last focused index path is focused automatically. 
 * If the collection view has never been focused, 
 * then the preferred focused index path is used.
 */
collectionV.remembersLastFocusedIndexPath = true

addSubview(collectionV)

......
```