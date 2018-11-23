---
title: '[Carthage怎么用]Carthage与CocoaPods对比及入坑指南'
date: 2018-04-12 10:12:36
tags:
---
# 前言
众所周知，多数iOS开发者在管理项目的三方框架时都在使用[CocoaPods](https://cocoapods.org "CocoaPods")，其简单的用法、良好的语言兼容性以及中心化的管理方式给项目开发带来了极大的便利。但随着项目逐步迭代，依赖的数量随之提升，CocoaPods的问题也逐渐暴露出来。

#### 批判前回顾下CocoaPods的优点
- 使用方便，除编写 Podfile 以外其他几乎都是自动完成
- 提供的三方框架数量巨大
- 静态/动态编译均支持

#### CocoaPods的问题在哪里？
- 每次更新环境都需要连接到中心仓库，比较耗时
- 使用xcworkspace构建与产品绑定的pod项目，每次pod install/pod update/xcode clean（Podfile.lock发生改变）都会重新编译依赖文件，极大的影响项目编译速度
- 为了兼容CocoaPods，框架作者有很多额外的工作

#### 那么Carthage就能解决问题？
可以，去中心化的管理方式不需要花费过多精力在依赖库的维护上，也不需要访问中心仓库读取配置，而且Carthage使用xcodebuild将依赖库事先编译为.framework文件后再引入项目，除非执行carthage update否则不会再次编译，也不会在Clean项目后重新编译所有依赖文件的代码。

#### Carthage优点
- 与CocoaPods 无缝集成，方便项目迁移
- 除了必要更新，不需要重复编译依赖文件
- 不需要访问中心仓库获取配置，节省时间
- 开发者为自己的框架添加Carthage支持异常简单（Xcode -> Project/Framework -> Set Scheme Shared）

#### 预知的风险
- 仍有很多库尚未支持Carthage
- 只支持动态库
- 需要自行管理依赖的接入

#### 说了这么多，CocoaPods和Carthage我到底该如何选择？
1. CocoaPods和Carthage可以很好的共存，互不影响，使用两者为项目添加的依赖也都可以彼此共享。
2. 如果项目有一些库是代码量比较大的，编译相对耗时，（在已经兼容Carthage的情况下）建议使用Carthage，这样避免了CocoaPods不必要编译带来的时间损耗
3. 如果项目有一些库需要经常更新，而文件体积又不是很大，推荐使用CocoaPods以获得最轻松的体验
4. 如果项目有一些库提供了子集（比如PromiseKit/CorePromise这样的形式）需要使用CocoaPods来集成，因为Carthage并不支持这样的特性，只会帮你把所有仓库拉取下来编译
5. 处女座强烈推荐CocoaPods！

# Carthage快速上手
1. 安装Cartfile：`brew update `、`brew install carthage`
2. 在项目根目录创建 [Cartfile](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#cartfile "Cartfile") 文件，以声明需要引入项目的三方库
3. 执行`carthage update --cache-builds --platform iOS`更新三方库的源代码，并编译为二进制文件
>该操作会在项目根目录下创建 [Carthage/Checkouts](https://github.com/Carthage/Carthage/blob/master/Documentation/Artifacts.md#carthagecheckouts) 文件夹，*Checkouts*放置从GitHub下载的依赖源码，*Build*文件夹放置编译后的二进制文件
4. 拖拽`.framework`文件到项目设置“*General*”选项卡的`Linked Frameworks and Libraries`中
5. 在项目`Build Phases settings`选项卡“*New Run Script Phase*”添加自定义脚本
{% codeblock %}
/usr/local/bin/carthage copy-frameworks
{% endcodeblock %}
检查依赖是否有更新（可选）
```
/usr/local/bin/carthage outdated --xcode-warnings
```
6. 在“Input Files”下添加framework的路径，例如：
```
$(SRCROOT)/Carthage/Build/iOS/Result.framework
$(SRCROOT)/Carthage/Build/iOS/ReactiveSwift.framework
$(SRCROOT)/Carthage/Build/iOS/ReactiveCocoa.framework
```
7. 在“Output Files”下添加framework的路径，例如：
```
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/Result.framework
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/ReactiveSwift.framework
$(BUILT_PRODUCTS_DIR)/$(FRAMEWORKS_FOLDER_PATH)/ReactiveCocoa.framework
```
>With output files specified alongside the input files, Xcode only needs to run the script when the input files have changed or the output files are missing. This means dirty builds will be faster when you haven't rebuilt frameworks with Carthage.

# 常见问题

**框架未链接到项目**
如果运行xcode后项目停留在断点，并在控制台展示如下信息：
```
dyld: Library not loaded: @rpath/框架名.framework/框架名
Referenced from: /Users/用户名/Library/Developer/CoreSimulator/Devices/...
Reason: image not found
```
则说明你没有将.framework正确链接到项目，请检查项目`Build Phases settings`选项卡“*Linked Frameworks and Libraries*”中已经拖入了所需要的.framework

# 更新
奈何我们项目过于庞大，有大量依赖是无法通过Carthage解决的，如果要强行接入Carthage则必须使用CocoaPods + Carthage混合的模式进行依赖管理，徒增了开发者的维护成本，最终决定放弃Carthage，使用CocoaPods进行集中管理。
