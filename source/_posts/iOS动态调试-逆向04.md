---
title: iOS动态调试[逆向04]
tags: 
	- iOS
	- 逆向
categories: 
	- iOS
	- 逆向
date: 2022-08-22 10:33:21
---
#### LLDB

最老版本的Xcode的调试器是GDB后来苹果就在这基础上开发了LLDB
现在LLDB都是内嵌在Xcode中
#### Xcode的调试原理
当我们在手机上连接Xcode进行真机调试的时候,Xcode识别手机后会将debug server安装到我们的手机上 (/Developer/usr/bin/debugserver)
> debugserver一开始是放在mac的Xcode中的`/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/DeviceSupport/9.0/DeveloperDiskImage.dmg/usr/bin/debugserver`
当我们进入某个断点时,手机上的`debugsever`会监听`lldb`传过来的指令
`debugserver`接收到指定的时候,再执行到我们的App上
接着App再把反馈信息传递给`debugserver` 然后`debugserver`再把消息反馈给`lldb`，`lldb`在把调试信息打印在Xcode上。

Xcode调试具有局限性，只能调试通过Xcode安装的App

#### 动态调试任意App

通过上述Xcode的调试原理要想调试App debugserver、lldb这两个必不可少 因为没有源码 我们只能在终端上代替上面的Xcode

##### step1.

Xcode自动给我们按照的debugserver肯定是用不了了,因为权限不够。所以我们需要给他签上权限

从手机上拖出debugserver（/Developer/usr/bin/debugserver）
重签名 – 将下列保存成entitlement.entitlements放在debugserver同级文件夹（其实就 是个xml文件 后缀名不重要）
```
<!DOCTYPE plist PUBLIC "-//Apple//DTD PLIST 1.0//EN"   "http://www.apple.com/DTDs/PropertyList-1.0.dtd"> 
<plist version="1.0"> 
<dict> 
 <key>com.apple.springboard.debugapplications</key> 
 <true/> 
 <key>get-task-allow</key> 
 <true/> 
 <key>task_for_pid-allow</key> 
 <true/> 
 <key>run-unsigned-code</key> 
 <true/> 
</dict> 
</plist>
```
使用`codesign`重签名`codesign -s – –entitlements ./entitlement.entitlements -f debugserver`
将签名完成的`debugserver`放入手机`usr/bin`目录下
并给权限
```
chomod +x /usr/bin/debugserver
```
这里不是之前替换原来的`debugserver` ,因为那个目录是只读的没有权限所以放在`/usr/bin/`下面

##### step2.

手机需要和mac做连接 详见SSH登录
```
python /Users/apple/Desktop/jailbreak/usbmuxd/tcprelay.py -t 22:10010 10011:10011(让手机的22、10011端口映射到本地的10010、10011端口)
```
登录手机
```
ssh root@localhost -p 10010
```
监听进程
```
debugserver *:10011 -a 【进程名】
```
接下来就可以在mac终端进入lldb 进入调试模式 （和在xcode上敲lldb指令一样）

#### debugserver 启动App
可观察App是如何启动的
```
debugserver  -x auto *:端口号 App的可执行文件路径 

```

