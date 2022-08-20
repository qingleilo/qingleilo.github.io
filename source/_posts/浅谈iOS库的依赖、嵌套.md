---
title: 浅谈iOS swift库 被依赖、被嵌套
date: 2022-08-03 10:16:00
tags: iOS
categories: iOS
---
## 浅谈iOS swift库 被依赖、被嵌套

说说sdk开发中 sdk嵌套swift库<或者oc、swift混编库> <下面以被嵌套的swift库名叫 `SwiftSdk.framework` 为例>

##### 集成方集成的时候各种报错

- 首先就会报sdk所依赖的swift库找不到`Could not find or use auto-linked framework 'SwiftSdk'`
  - 解决方法1:为集成方提供`SwiftSdk` 。
    - 这里还会有个问题：打包`SwiftSdk`的xcode所用的swift版本不同 报错类似这种`Module compiled with Swift 5.5 cannot be imported by the Swift 5.5.2 compiler: `网上也有些解决方法，下载工具链什么的。 太麻烦， 把你的`SwiftSdk.framework`转换成xcframework 可以完美解决。
  - 解决方法2：不管这个`SwiftSdk`是静态还是动态，改成`Embed` 。下一步 让集成方在`build setting`->`Framework search Path`加上`SwiftSdk`的路径。 (前面改成embed后，打包的sdk里面的`Frameworsks`文件夹会有`SwiftSdk`)。若还是报错，添加`run search path`路径

如果集成方是纯oc还会有下面的报错

- 又出现依赖的swift系统库 找不到`Could not find or use auto-linked framework`
  - 解决方法1:集成方创建swift文件自动建立桥接
  - 解决方法2:手动创建桥接、`build setting `--> `Add User-Defined Setting` -->`SWIFT_VERSION`=5

总结：无论swift库是动态库还是静态库，只要被嵌套，在外面都出现链接错误。



另外说说 `Podfile`文件 中`use_frameworks!` 、 `use_modular_headers!` 以及 `:modular_headers => true`

> `.podspec` 参考 https://guides.cocoapods.org/syntax/podspec.html 
>
> `.podfile` 参考 https://guides.cocoapods.org/syntax/podfile.html

在制作swift或者混编私有库的时候为`.podspec`添加`  s.static_framework = true`

可以让你全局使用`use_frameworks!`  时， `:modular_headers => true`指定让你的库以静态库方式加载











