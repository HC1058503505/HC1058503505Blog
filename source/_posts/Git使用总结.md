---
title: Git使用总结
date: 2017-06-07 09:41:22
tags: Git
categories: Git
---
> Git的命令的使用，上传代码到远程与更新远程代码到本地仓库. <!--more-->

## 使用Git上传代码到远程库

- 建立git仓库
```yml
cd到你的本地项目根目录下，执行git命令

git init
```

- 将项目的所有文件添加到仓库中
```yml
git add .
```
 如果想添加某个特定的文件，只需把.换成特定的文件名即可


- 将add的文件commit到仓库
```yml
git commit -m "注释语句"
```

- 去github上创建自己的Repository，拿到仓库地址https


- 将本地的仓库关联到Github上
```yml
git remote add origin https://仓库地址
移除仓库：git remote rm origin
```
- 上传github之前 要先pull一下 执行如下命令
```yml
git pull --rebase origin master
```

- 上传代码到github远程仓库
```yml
git push -u origin master
```

## 更新远程代码到本地仓库

### 方式一
- 查看远程仓库
```yml
$ git remote -v

origin	https://github.com/HC1058503505/HCCardView.git (fetch)
origin	https://github.com/HC1058503505/HCCardView.git (push)
```

- 从远程获取最新版本到本地
```yml
// 从远程的origin仓库的master分支下载代码到本地的origin master
$ git fetch origin master
```

- 比较本地常看远程仓库的区别
```yml
$ git log -p master.. origin/master

// 结果如下
commit 4d89094a5b01e29ab6f5f2e8bfbe5c944059840b
Author: HC1058503505 <1058503505@qq.com>
Date:   Tue Jun 6 17:58:11 2017 +0800

    Update README.md

diff --git a/README.md b/README.md
index cd16927..7907a33 100644
--- a/README.md
+++ b/README.md
@@ -136,8 +136,89 @@ extension HCCardHeaderView:HCCardContentViewDelegate {
 
 * UICollectionView的自定义布局
- > 自定义布局的实现步骤
- 1.// MARK: - 准备布局
- 2.// MARK: - 返回布局
- 3.// MARK: - 布局作用范围
+
+自定义布局的实现步骤
+1.// MARK: - 准备布局
+extension HCCollectionViewFlowLayout {
+    override func prepare() {
```

- 远程与本地仓库合并
```yml
$ git merge origin/master
```

### 方式二
- 查看远程分支，和方式一相同
- 从远程获取最新版本到本地
```yml
// 从远程的origin仓库的master分支下载到本地并新建一个分支temp
$ git fetch origin master:temp
```
- 比较本地仓库和远程仓库的区别
```yml
$ git diff temp
```
- 合并temp分支到master
```yml
$ git merge temp
```
- 如果不想要temp分支，可以删除
```yml
git branch -d temp
```

参考链接：
[git fetch 的简单用法:更新远程代码到本地仓库](http://www.cnblogs.com/zknublx/p/6113667.html)