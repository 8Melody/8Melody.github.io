---
layout: post
title: "Git常用操作汇总"
subtitle: "Git operators"
author: "XYH"
header-img: ""
header-bg-css: "linear-gradient(to right, #404040, #687a86);"
tags:
  - Git
---

> 本文记录常用Git操作。

# git常用操作汇总

## 创建git仓库
> git init


之后会出现
> Initialized empty Git repository in C:/Users/xxx/.git/

在当前目录会成功创建`.git`目录。

## 配置用户信息
 > git config --global user.email 'xxx@xxx.com' 
 > git config --global user.name 'xxx'
  
## clone 
> git clone xxxx.git

## 提交
> git add 文件名(如果添加所有修改的文件直接git add .)
> git commit -m '提交信息'
  
## 本地分支推送到远程分支
> git push origin 分支名称

## 暂存
> git stash

## 恢复暂存
> git stash pop

## 查看本地分支
> git branch 

## 查看所有分支
> git branch -a 

## 创建本地分支
> git branch 分支名

## 创建远程分支
> git push origin 分支名

## 合并本地分支
> git merge 分支名

## 合并远程分支
切换到对应的本地分支，先合并本地分支，然后推送到远程分支
> git checkout 本地分支
> git merge 需要合并的本地分支
> git push origin 远程分支

## 删除本地分支
> git branch -d 分支名称

## 删除远程分支
> git push origin --delete 分支名称

## 切换分支
> git checkout 分支名

## 创建同时切换分支
> git checkout -b bug-fix

在当前分支上创建分支`bug-fix`，并且换到`bug-fix`

## 查询所有tag
> git tag

## 打tag
> git tag xx -m 'xxx'
> git push origin --tags (推送到托管服务器)

## 删除本地tag
> git tag -d tag名称

## 删除远程tag
> git push origin --delete tag tag名称

## 拉取远程分支
> git pull origin v1.2.0

将远程分支`v1.2.0`拉取到本地分支，在拉取之前如果本地有修改没有提交，可以先暂存`git stash`，拉取完毕之后`git stash pop`，如果出现冲突，可以输入`git mergetool`使用图形化解决冲突。

## git checkout 远程分支
> git checkout -b 本地分支名 origin/远程分支名

可以在远程仓库基础上拉取指定分支。

## git fetch 
`git fetch` 和 `git pull` 很像，但 `git fetch` 要柔和一点，`git pull` 会将远程的命令拉下来直接和本地的合并。

`git fetch` 会先获取信息，但不合并。

> git fetch origin dev
> git log -p dev origin/dev(比较本地版本跟远程版本)
> git merge origin/dev(合并远程dev)

或者：
> git fetch origin dev:localBranch

 将远程`dev`分支拉取下来，并同步到 `localBranch` 分支上，如果 `localBranch` 分支不存在则创建，但不会切换。


## 版本回退
> git reset --hard 

会回退到最近一次的提交（HEAD）

如果想回退到指定的提交的话需要指定提交需要先查到提交的id

> git log 

查询到commit的id

> git reset --head 版本号 

这时候如果再push则会报错，因为本地的内容比远程内容版本低。
这个时候可以选择强推，要负责强推的内容无问题，后果自负。

> git push origin xxx -f

## 重新设置远程仓库

> git remote rm origin

添加新的远程仓库

> git remote add origin xxx.git


