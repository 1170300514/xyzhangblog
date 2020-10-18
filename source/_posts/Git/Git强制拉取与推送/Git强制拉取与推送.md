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



TODO: 解决git冲突