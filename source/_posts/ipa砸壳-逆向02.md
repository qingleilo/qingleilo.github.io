---
title: ipa砸壳[逆向02]
tags: 
	- iOS
	- 逆向
categories:
	- iOS
	- 逆向
date: 2022-08-22 10:20:59
---

#### 验证可执行文件是否已经脱壳
- 使用MachOView软件查看，Load Commands -> LC_ENCRYPTION_INFO->Crypt ID的值 0代表未加密（已脱壳）
- 命令行工具` otool -l 可执行文件路径 ｜ grep crypt`

#### 砸壳工具一[CrackerXI App]
- Cydia添加源http://apt.wxhbts.com/
- 安装CrackerXI
- 打开CrackerXI+选择你要砸壳的App
- 成功后会保存在/var/mobile/Documents/CrackerXI路径下