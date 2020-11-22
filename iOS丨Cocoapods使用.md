---
title: iOS丨Cocoapods使用
date: 2020-11-18 22:21:12
categories:
    - iOS
tags: 
    - iOS
mathjax: true
---

## 背景
之前使用pod也是简单的进行pod install之类的操作，并没有深入看一看使用pod之后发生的若干改变。这一次恰好要额外手动引入一个第三方库，并且和目前已经在pod里面的库是重名的。记录一下碰到的问题。也把cocoapod这个工具整理一下。

## Pod结构
引入pod首先要定义podfile。这个本质上是一个config文件，以声明的方式确定需要引入哪些库。
```swift
# Uncomment the next line to define a global platform for your project
source 'https://github.com/CocoaPods/Specs.git'
platform :ios, '11.0'

target 'Mercury' do
  # Comment the next line if you don't want to use dynamic frameworks
  #use_frameworks!

  # Pods for Mercury
  pod 'SnapKit', '~> 5.0.0'
  pod 'Alamofire', '~> 5.2'
end

```
之后直接`pod install`就可以直接安装这些库了。安装好了之后目录结构会发生变化：
- 首先是生成了一个workspace。xcworkspace文件，使用CocoaPod管理依赖的项目，XCode只能使用workspace编译项目，如果还只打开以前的xcodeproj文件进行开发，编译会失败。而xcworkspace文件实际是一个文件夹，实际Workspace信息保存在contents.xcworkspacedata里，该文件的内容非常简单，实际上只指示它所使用的工程的文件目录
- podfile.lock是当前安装依赖库的版本
![](iOS%E4%B8%A8Cocoapods%E4%BD%BF%E7%94%A8/%E6%88%AA%E5%B1%8F2020-11-21%20%E4%B8%8A%E5%8D%8810.01.56.png)

关于pods的更多内容可以看这篇文章[Cocoapods原理总结 - iOS](https://juejin.cn/entry/6844903502829846541)

## 引入第三方库
### pods引入
pods引入是非常简单的。我们只需要在podfile中声明即可。

### 手动引入
但有的时候使用pod并不能解决问题。比如我们可能需要同一个库的两个版本，以lottie为例。使用podfile的话只能引入一个版本，如果是两个的话生成的target实际上会重复。因此pod只会安装位置在前面的那一个。
![](iOS%E4%B8%A8Cocoapods%E4%BD%BF%E7%94%A8/%E6%88%AA%E5%B1%8F2020-11-21%20%E4%B8%8A%E5%8D%8810.21.09.png)

我们可以直接把代码拖进去，但是可能平时开发使用的代码本身就是通过pod管理的。本地手动拖的话一次pod install就又都没有了。因此还是要通过pod管理一下。

### 建一个本地的pod库
我们可以通过建一个本地库的方式去完成这个操作。
```swift
#
# Be sure to run `pod lib lint BNShare.podspec' to ensure this is a
# valid spec before submitting.
#
# Any lines starting with a # are optional, but their use is encouraged
# To learn more about a Podspec see https://guides.cocoapods.org/syntax/podspec.html
#

Pod::Spec.new do |s|
  s.name             = 'LottieOC'
  s.version          = '0.0.1'
  s.summary          = 'LottieOC'
  s.description      = <<-DESC
M Team LottieOC
                       DESC
  s.homepage         = 'https://code.byted.org/anote/TTBeethoven'
#  s.license          = { :type => 'MIT', :file => 'LICENSE' }
  s.author           = { "" => "" }
  s.source           = { :git => 'ssh://no_such_git_yet/haha.git', :tag => s.version.to_s }
  s.ios.deployment_target = '10.0'
  s.source_files = '**/*.{h,m,swift}'
#  s.vendored_libraries = 'xx/Classes/Vendor/SQLcipherArchive/xx.a'
#  s.libraries = 'sqlite3'
  # s.resource_bundles = {
  #    'TTBeethovenNetModel' => ['Assets/*', "**/*.xib"]
  #   }
  #s.prefix_header_contents = '#import "TTFoundation.h"'

#  s.public_header_files = '**/*.h'
  
#  non_arc_files = 'eif-iOS-framework/Classes/HDFoundation/HDZombie/NSObject+Zombie.m','eif-iOS-framework/Classes/HDFoundation/HDZombie/HDZombieAdvanceHelper.m'
#  s.exclude_files = non_arc_files
#  s.subspec 'TTNonArc' do |sp|
#   sp.source_files = non_arc_files
#   sp.requires_arc = false
#  end
  
  s.frameworks = 'Foundation', 'UIKit'
  
end

```
接着在podfilie中去引入这个库，注意source_path一定要选择本地的文件夹。

接下来执行pod install，就会发现这个库可以被pod进来了。并且module的名称是podspec中定义的名称。

需要注意的一点是，在新生成的pod文件中，一定要引入一个顶级的swift文件，名称为module的名称。否则是不会生成对应的modulemap的。

在别的pod中去引入目前的pod时，也需要在对应的podspec中引入dependcy。

