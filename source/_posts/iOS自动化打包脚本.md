---
title: iOS自动化打包脚本
tags: iOS
categories: iOS
date: 2022-08-20 17:50:57
---

> 闲来无事学习一下shell

#### 打包xcframework

- 由于xcode13的模拟器也包含arm64架构，移除模拟器arm64架构，`build Setting`=>`Excluded Architectures`=>勾选`Any ios simulator sdk` 添加`arm64` 也可在脚本中设置-arch

```
#!/bin/bash
set -e
set -u
set -o pipefail

SCHEME_NAME=sdk
WORKSPACE_NAME=sdk
#打包环境
CONFIGURATION=Release
ROOT_PATH=$(pwd)
#输出路径
EXPORT_PATH=${ROOT_PATH}/archive


#chech scheme
xcodebuild -list || exit 0

#clean
function arch_clean {
    xcodebuild clean \
    -workspace ${WORKSPACE_NAME}.xcworkspace \
    -scheme ${SCHEME_NAME} \
    -configuration ${CONFIGURATION}
}

#下面iphonesimulator15.5 可根据 xcodebuild -showsdks查看
 # #打包模拟器
function archive_simulator {
     xcodebuild archive \
     -workspace ${WORKSPACE_NAME}.xcworkspace \
     -scheme ${SCHEME_NAME} \
     -destination="iOS Simulator" \
     -sdk iphonesimulator15.5 \ 
     -configuration ${CONFIGURATION} \
     SKIP_INSTALL=NO \
     BUILD_LIBRARIES_FOR_DISTRIBUTION=YES \
     -archivePath ${EXPORT_PATH}/ios_simulator.xcarchive
 }

function archive_iosDevice {
     xcodebuild archive \
     -workspace ${WORKSPACE_NAME}.xcworkspace \
     -scheme ${SCHEME_NAME} \
    -sdk iphoneos \
     -destination="iOS" \
     -configuration ${CONFIGURATION} \
     SKIP_INSTALL=NO \
     BUILD_LIBRARIES_FOR_DISTRIBUTION=YES \
     -archivePath ${EXPORT_PATH}/ios_device.xcarchive
 }

 # #创建xcframework
 function create_xcframework {
     xcodebuild -create-xcframework \
     -framework ${EXPORT_PATH}/ios_device.xcarchive/Products/Library/Frameworks/${SCHEME_NAME}.framework \
     -framework ${EXPORT_PATH}/ios_simulator.xcarchive/Products/Library/Frameworks/${SCHEME_NAME}.framework \
     -output ${EXPORT_PATH}/${SCHEME_NAME}.xcframework
 }


 arch_clean || exit 0
 archive_simulator || exit 0

 if [[ -e ${EXPORT_PATH}/ios_simulator.xcarchive ]] ;then
     echo -e "\033[1135m🍺🍺🍺🍺🍺🍺xcarchive ios_simulator success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌xcarchive ios_simulator fail!❌❌❌❌❌❌ \033[0m"
 fi

archive_iosDevice || exit 0

if [[ -e ${EXPORT_PATH}/ios_device.xcarchive ]] ;then
     echo -e "\033[1135m🍺🍺🍺🍺🍺🍺xcarchiveios_device success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌xcarchive ios_device fail!❌❌❌❌❌❌ \033[0m"
 fi

 cd ${EXPORT_PATH} 
 create_xcframework || exit 0
 if [[ -e ${EXPORT_PATH}/${SCHEME_NAME}.xcframework ]] ;then
    echo -e "\033[1135m🍺🍺🍺🍺🍺🍺create ${SCHEME_NAME}.xcframework success!🍺🍺🍺🍺🍺🍺 \033[0m"
 else
     echo -e "\033[1135m❌❌❌❌❌❌create ${SCHEME_NAME}.xcframework fail!❌❌❌❌❌❌ \033[0m"
 fi

 open .

```

####  打包ipa
