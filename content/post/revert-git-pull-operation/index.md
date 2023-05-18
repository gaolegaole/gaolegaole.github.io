---
title: "恢复git pull操作"
date: 2023-05-18T10:26:11+08:00
draft: false
description: "恢复git pull操作"
categories:

tags:
    - git
---
> 背景：在生产git pull的时候，由于长时间没有操作，导致输入了：git pull repo master，但是生产的分支是pre-production。并且这其中还有一些内容是有冲突的。导致pull了一些错误内容。

# 解决方法
## 1. 执行git relog查看历史变更
```shell
[root@VM]# git reflog
3d3da9f0 (HEAD -> pre-production, repo/pre-production) HEAD@{0}: pull repo master
0a06930d HEAD@{1}: pull repo pre-production: Fast-forward
ba745252 HEAD@{2}: pull repo pre-production: Fast-forward
9dad35fb HEAD@{3}: pull repo pre-production: Fast-forward
...
```
## 2. 然后用git reset --hard HEAD@{n}，（n是你要回退到的引用位置）回退。
```shell
git reset --hard HEAD@{1}
```
这样就回退到上一次了，`{0}`是当前的位置。也可以使用：`git reset --hard 0a06930d` 进行回退。