---
layout: post
title: PostgreSQL 入门
description: 写了半天，电脑蓝屏了，内容全部变成NUL了，就这样
category: post
tags: database postgresql pg
published: true
lastUpdate: 2015-10-27
---

## 安装 ##
被蓝屏搞没了
默认的initdb数据目录是 `/var/lib/pgsql/data/`

## 简介 ##
被蓝屏搞没了

## 入门 ##
> 重写

### 创建数据库 ###
pg的管理模式和mysql很不一样。在MySQL我们要登陆到mysql服务器才能进行创建数据库，添加用户等操作。但是pg提供了一个`createdb` 的命令，可以直接在bash中创建数据库。但是要求： 运行该命令的用户必须是启动postgresql服务的用户。

使用yum安装的pg会自动创建一个postgres用户并用这个用户启动相关的各个线程，所以要运行`createdb` 命令，需要先切换到postgres 用户。由于是系统创建的用户，我们也不知道用户的密码是什么，可以先重置 postgres 的密码。

```bash
sudo passwd postgres
// enter new password here

su postgres
createdb golang
// dropdb golang
```
当然也可以使用命令参数指定用户

```
createdb -U postgres golang
```
但是运行并没有成功，还是没有权限，切换到postgres用户比较好用。这样的管理方式对于习惯mysql的人来说，多少有些奇怪。

### 创建pg用户 ###
在登陆管理pg之前我们要学会怎么创建pg的用户，并用这个用户去登陆pg的服务。
说实话 [官方文档](http://www.postgresql.org/docs/9.2/interactive/auth-username-maps.html) 看起来可不是那么简单。

### psql ###
