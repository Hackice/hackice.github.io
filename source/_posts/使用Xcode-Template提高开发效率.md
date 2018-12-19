---
title: 使用Xcode Template提高开发效率
date: 2018-12-18 15:55:55
tags: Xcode
---
Template，又分为Project Template和File Template，是Xcode创建新项目或文件所使用的“模版”，我们每次开发一个新的功能都会使用到它，没有谁会愿意每次都从文件头写到默认实现。结合实际的使用场景，来看看如何自定义两种常用的文件Template。
<!--more-->
#### 情景一
项目中想要新建一个模型，需要创建一个.swift文件、声明依赖、实现模型、实现模型转换或网络层所需的协议，这个过程中其实只有模型名称及其属性是需要我们自己去写，其他代码都可以使用Template来实现。

#### Template路径
Xcode默认提供了非常多的Template，放在`Xcode.app/Contents/Developer/Library/Xcode/Templates`目录下，比如我们常用的Swift文件模版就放在File Templates/Source/Swift File.xctemplate中。

虽然我们可以直接修改这些Xcode默认提供的Templates，但其实Xcode也会到`~/Library/Developer/Xcode/Templates/File Templates`中去查找是否有自定义模版，同时在**每次启动Xcode**时将他们加载。

#### 如何创建Template
其实从Xcode默认提供的模版当中，我们可以看出一个Template其实是一个以`.xctemplate`为后缀的文件夹，它包含了使用固定名称的几个配置文件：
- `TemplateIcon.png`及其@2x文件为模版选择窗中显示的图标
- `TemplateInfo.plist`为模版创建时所需要的配置
- `___FILEBASENAME___.swift`就是我们希望模版能够创建出的特定格式的代码

好了，那么我们结合上面提到的情景，创建一个“模型文件”的File Template：
1. 我们沿用swift文件模版的默认配置，直接将
`Xcode.app/Contents/Developer/Library/Xcode/Templates/File Templates/Source/Swift File.xctemplate`
拷贝至`~/Library/Developer/Xcode/Templates/File Templates/Custom/`
并重命名为`Model.xctemplate`
2. 修改`___FILEBASENAME___.swift`为“模型文件”的通用代码：
```swift
//___FILEHEADER___

import ObjectMapper

struct ___FILEBASENAMEASIDENTIFIER___: Mappable, ResponseObjectSerializable {
    
    var <#name#>: <#type#>?
    
    init?(map: Map) {}
    
    mutating func mapping(map: Map) {
        <#name#> <- map["<#name#>"]
    }
}
```
3. 重启Xcode即可

至此一个最简单的Template就创建好了，你可以在Xcode中使用快捷键Command+N创建文件窗口中找到刚才创建的Model模版。

#### Template中的标识符
Template中有很多实用的预定义标识符，比如刚才我们创建的Model模版中就用到了文件头标识``___FILEHEADER___``，用来创建文件头部信息，效果如下：
```swift
//
//  DemoDataSource.swift
//  Project
//
//  Created by Marcus Wang on 2018/12/18.
//  Copyright © 2018年 Company. All rights reserved.
//
```
也可以使用特定关键字标识定制需要的效果：

| 关键字 | 释义 | e.g. |
| :------: | :------: | :------: |
| ``___FILENAME___`` | 文件名 | DemoDataSource.swift |
| ``___FILEBASENAMEASIDENTIFIER___``或``___FILEBASENAME___`` | 类名 | DemoDataSource |
| ``___PROJECTNAME___`` | 项目名 | Project |
| ``___FULLUSERNAME___`` | 用户名 | Marcus Wang |
| ``___DATE___`` | 日期 | 2018/12/18 |
| ``___YEAR___`` | 年份 | 2018年 |
| ``___ORGANIZATIONNAME___`` | 组织名称 | Company |
| **高阶用法** |
| ``___VARIABLE_xxx___`` | 配置选项关键词 | xxx所指向的选项名称，可以是某个类名或任何东西 |

#### 情景二（高阶用法）
项目中想要新建一个网络请求，单独使用上述文件模版并不能完全实现即开食用，因为网络请求类型比如GET/POST/PUT/DELETE，或者接口响应是对象还是列表，并不是使用单一格式就能解决的，因此我们需要一个提供配置选项的Template。

同样，参考Xcode默认的`Cocoa Class.xctemplate`，我们在`~/Library/Developer/Xcode/Templates/File Templates/Custom/`中新建一个`DataSource.xctemplate`，并在其文件夹中使用子文件夹的形式创建出所需要支持的所有网络请求，例如我创建了DataSource（对象）、ArrayDataSource（单页列表）、NextableArrayDataSource（分页列表），每个子文件夹中都需要放置对应的代码模版文件`___FILEBASENAME___.swift`，例如DataSource的为：
```swift
//___FILEHEADER___

struct ___VARIABLE_dataSourceName___: ___VARIABLE_dataSourceSubclass___ {
    
    typealias T = <#ObjectMapper Model#>
    
    let endPoint: URLRequestComponent
    init(<#parameters#>: <#parameters type#>) {
        endPoint = <#RModel#>(<#parameters#>: <#parameters type#>)
    }
    // otherwise, if no parameters you can write like this:
    // let endPoint: URLRequestComponent = <#RModel#>()
}

private struct <#RModel#>: URLRequestComponent {
    
    fileprivate let <#parameters#>: <#parameters type#>
    
    init(<#parameters#>: <#parameters type#>) {
        self.<#parameters#> = <#parameters#>
    }
    
    var url: HTTPURLStringConvertible {
        return <#URIV3#>(path: <#url path string#>)
    }
    
    var parameters: [String: Any] {
        return ["fields": fields]
    }
    
    var fields: String = <#"{id}"#>
}
```
最后修改Template的配置选项文件`TemplateInfo.plist`，其中`Options`字段用来声明创建选项，例如：
```xml
<dict>
   <key>Identifier</key>
   <string>dataSourceName</string>
   <key>Required</key>
   <true/>
   <key>Name</key>
   <string>Class:</string>
   <key>Description</key>
   <string>The name of the class to create</string>
   <key>Type</key>
   <string>text</string>
   <key>NotPersisted</key>
   <true/>
</dict>
```
其中`Identifier`对应前面列举的预定义标识符中的`___VARIABLE_xxx___`，在代码中我们该条选项的名称为`___VARIABLE_dataSourceName___`，模版文件可以通过该标识获取到创建时所配置的信息。

最后，重启Xcode，就可以在模版选择窗口中看到DataSource了，现在我们会看到多出了一个选项菜单用于选择网络请求的类型。

#### 结语
与代码块相比，Xcode并没有提供一个便捷的创建Template的方式，因此在实际的开发场景中我们可能更多的是使用默认模版创建，并在其基础上进行修改，为什么不让这个过程变得更酷一点呢？

本文提及的2个.xctemplate都已经上传至[GitHub](https://github.com/Hackice/XcodeFileTemplate ".xctemplate source files")。