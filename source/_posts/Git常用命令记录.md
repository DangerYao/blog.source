---
title: Git常用命令记录
date: 2018-09-05 19:28:35
categories:
 - technology
tags:
 - technology
 - notes
 - git
---

##### 将目录初始化为一个git项目
```shell
git init
```
会在目录里面生成一个.git的隐藏目录

##### 将所有文件放进新的本地git仓库
```shell
git add .
```
如果你本地已经有 .gitignore 文件，会按照已有规则过滤不需要添加的文件。如果不想要添加所有文件，可以把 . 符号换成具体的文件名

##### 将添加的文件提交到仓库
```shell
git commit -m message
```

##### 将本地仓库关联到远程仓库
```shell
git remote add origin url
```

##### 删除本地仓库和远程仓库的联系
```shell
git remote remove origin
```

##### 查看git状态
```shell
git status
```

##### 提交代码到远程仓库
```shell
git push orign master
```
可以将master换成对应的分支
##### 更新远程仓库代码
```shell
git pull orgin master
```

*未完待续*


