---
title: Git强制拉取与推送
date: 2020-10-18 15:52:13
tags: 
    - git
summary: 使用Git遇到的一些与解决冲突相关的操作 TODO：解冲突方法
---



## Git强制拉取

有时本地修改代码出现异常，需要将本地修改全部忽略，强制拉取远端

```bash
// 方法1: 清空本地修改后 拉取远端代码
git reset --hard
git pull

// 方法2: 直接拉取远端代码将HEAD指向master最新版本
git fetch --all
git reset --hard origin/master
git pull
```

## Git强制推送

强制用本地代码覆盖远程仓库

```bash
git push -f origin master
```

## Git撤销commit

执行完commit后，想撤回commit，以下命令可以仅撤回commit不删除代码

```bash
git reset --soft HEAD^
```

#### 回滚次数

HEAD^的意思是上一个版本，也可以写成HEAD~1

如果你进行了2次commit，想都撤回，可以使用HEAD~2

#### 参数含义

1. --mixed 

   意思是：不删除工作空间改动代码，撤销commit，并且撤销git add . 操作

   这个为默认参数,git reset --mixed HEAD^ 和 git reset HEAD^ 效果是一样的。

2. --soft 

   不删除工作空间改动代码，撤销commit，不撤销git add . 

3. --hard

   删除工作空间改动代码，撤销commit，撤销git add . 

   注意完成这个操作后，就恢复到了上一次的commit状态。

#### 仅修改commit注释

```bash
git commit --amend
```

此时会进入默认vim编辑器，修改注释完毕后保存就好了。



TODO: 解决git冲突