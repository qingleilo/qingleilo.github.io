---
title: ssh登录iphone[逆向01]
tags: 
	- iOS
	- 逆向
categories: 
	- iOS
	- 逆向
date: 2022-08-22 09:58:00
---

在iOS逆向工程中，经常需要通过命令行来对iphone下达指令。

> iOS下有两个常用的账户：root、mobile 
> root:最高权限
> mobile:普通权限，只能操作一些普通文件
> iOS设备对外提供ssh服务的端口号是22
> iOS越狱设备通过Cydia安装OpenSSH工具

#### 无线模式

这种模式需要保证iOS设备和mac在同一局域网下

在mac终端输入$ssh 账户@ip地址 即可（初始密码：alpine 后续修改密码等详见Cydia中[OpenSSH 访问教程]、[Root 密码教程]）

```
ssh root@127.0.0.1
```

#### 有线模式
该种方式采用的是Mac上的一个服务程序usbmuxd(/System/Library/PrivateFrameworks/MobileDevice.framework/Resources/usbmuxd)

原https://cgit.sukimashita.com/usbmuxd.git/snapshot/usbmuxd-1.0.8.tar.gz在macos 12.3+ 上不适用 这里说另外一个iproxy

##### 安装usbmuxd工具包
```
brew install usbmuxd
```


##### 映射iphone22端口到mac并监听
<img src="listen.png" width = "100%" />

```
iproxy 10010 22
```

##### 登录iOS设备
不要关闭step2的窗口，command+T 另开个终端界面
```
ssh -p 10010 root@localhost
//或
//ssh root@127.0.0.1 -p 10010
```
##### 退出登录
```
exit
```
#### 免密登陆
##### 创建密钥
```
ssh-keygen
```
##### 将生成的公钥追加到授权文件尾部（这里如果iOS 设备没有.ssh 文件夹，需要手动创建）
```
scp -P 10010 ~/.ssh/id_rsa.pub root@localhost:~/.ssh
```
##### 登录iOS设备执行
```
cat ~/.ssh/id_rsa.pub >> authorized_keys
rm  ~/.ssh/id_rsa.pub
```
退出设备再次登录就不需要密码了


