---
title: 创建pod私有库
date: 2020-03-15 17:00:47
tags: 
    - Cocoapods
    - pod

categories:
    - 杂记
---

## 前言
看本地cocoaPod库 `cd ~/.cocoapods/repos`在repos文件夹下面，可以看到你当前存在的一些库，当你创建了自己的私有仓库时，也会在这里显示。

按照正常逻辑来看，私有库是一个地址，版本号管理库是一个地址，最好不要把两者放在一起，会显得很乱。
所以创建私有库，我们需要创建两个库：一个是私有仓库repo，用来做私有库的版本管理；另一个是私有代码库，用来做代码编写。

## 创建私有仓库 repo
这里的私有空间是指私有库存放的位置，repo是repository（存储库）的缩写，如果私有库的podSpec和代码存放在一起就会显得很乱，所有通常的做法就是再私有空间只存放私有库的podSpec文件（可以理解为版本号）。
### 创建远程私有repo
这里使用码云，可以免费创建多个私有工程。例如创建了一个私有存储库：https://gitee.com/devAlan/PrivateRepo。（已不存在）
### 添加到本地仓库中
`pod repo add [本地仓库repo名称] [远程repo地址]`
`pod repo add PrivateRepo https://gitee.com/devAlan/PrivateRepo`
使用上面步骤就可以将远程的私有版本存储仓库添加到本地。在Finder中查看repos中就可以发现增加了一个文件夹PrivateRepo。

## 创建私有代码库

创建代码私有库有两种方式，第一种是我们正常的使用Xcode创建项目的过程，另一种是使用pod模板来创建的过程，推荐使用模板来创建。

### 创建一个私人代码库
创建一个私人代码库，在创建时添加MIT License和README。
使用模板创建时，创建空仓库。

#### 将仓库clone到本地，添加项目工程文件。
#### 添加.podspec文件
“.podspec”文件是这个代码库的pod描述文件，使用pod命令创建空白模板。<br>
`pod spec create 项目名称`<br>
这里使用上面的私有代码库创建`pod spec create privateAdd`。

需要注意一下：生成的默认的podspec文件中，所有没有注释掉的都必须填写正确。

### 使用pod模板创建

```
pod lib creare [项目名称]（最好与远端一致）

// 以下为模板创建时，会用到的一些选项问题，根据实际选择就好。
What platform do you want to use?? [ iOS / macOS ]
 > iOS

// 选择编程语言
What language do you want to use?? [ Swift / ObjC ]
 > ObjC

/// 是否需要添加本地库
Would you like to include a demo application with your library? [ Yes / No ]
 > No

// 是否有frameworks
Which testing frameworks will you use? [ Specta / Kiwi / None ]
 > None

//
Would you like to do view based testing? [ Yes / No ]
 > No

// 创建的文件前缀
What is your class prefix?
 > HM
```
上述命令操作完成之后，会生成相应的模板文件。
代码文件都存放在`/[项目名称]/class/`下。
图片都存放在`/[项目名称]/Assets/`下。

### 将本地 Lib 工程与远程私有 lib Git 仓库关联 

```
git remote add origin 远程仓库地址  
    git错误处理
    git pull --rebase origin master 
    git push -u origin master           
    
git push origin master   #同步操作
    文件冲突 （保留一份 license 和 readme）
    git pull origin master --allow-unrelated-histories
    git mergetool
```

## 编辑“.podspec”文件


```python
#
#  Be sure to run `pod spec lint PrivateAdd_v2.podspec' to ensure this is a
#  valid spec and to remove all comments including this before submitting the spec.
#
#  To learn more about Podspec attributes see http://docs.cocoapods.org/specification.html
#  To see working Podspecs in the CocoaPods repo see https://github.com/CocoaPods/Specs/
#

# 声明
Pod::Spec.new do |s|

  # ―――  Spec Metadata  ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  These will help people to find your library, and whilst it
  #  can feel like a chore to fill in it's definitely to your advantage. The
  #  summary should be tweet-length, and the description more in depth.
  #
  # 私有库名称
  s.name         = "PrivateAdd_v2"    
  # 版本号，跟当前版本tag有关
  s.version      = "0.0.1"            
  # 项目描述
  s.summary      = "A short description of PrivateAdd_v2." 

  # This description is used to generate tags and improve search results.
  #   * Think: What does it do? Why did you write it? What is the focus?
  #   * Try to keep it short, snappy and to the point.
  #   * Write the description between the DESC delimiters below.
  #   * Finally, don't worry about the indent, CocoaPods strips it!
  s.description  = <<-DESC
                   DESC

  # 主页，自动生成的podspec文件中所有的EXAMPLE都需要改
  s.homepage     = "http://EXAMPLE/PrivateAdd_v2" 
  # 项目截图
  # s.screenshots  = "www.example.com/screenshots_1.gif", "www.example.com/screenshots_2.gif"  


  # ―――  Spec License  ――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Licensing your code is important. See http://choosealicense.com for more info.
  #  CocoaPods will detect a license file if there is a named LICENSE*
  #  Popular ones are 'MIT', 'BSD' and 'Apache License, Version 2.0'.
  #

  # 项目遵守的协议
  s.license      = "MIT (example)"      
  # s.license      = { :type => "MIT", :file => "FILE_LICENSE" }


  # ――― Author Metadata  ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Specify the authors of the library, with email addresses. Email addresses
  #  of the authors are extracted from the SCM log. E.g. $ git log. CocoaPods also
  #  accepts just a name if you'd rather not provide an email address.
  #
  #  Specify a social_media_url where others can refer to, for example a twitter
  #  profile URL.
  #

  # 作者信息
  s.author             = { "liujiajia" => "liujiajia@gerrit.babytree-inc.com" }  
  # Or just: s.author    = "liujiajia"
  # s.authors            = { "liujiajia" => "liujiajia@gerrit.babytree-inc.com" }
  # s.social_media_url   = "http://twitter.com/liujiajia"

  # ――― Platform Specifics ――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  If this Pod runs only on iOS or OS X, then specify the platform and
  #  the deployment target. You can optionally include the target after the platform.
  #

  # s.platform     = :ios           # 平台
  # 平台以及版本号
  # s.platform     = :ios, "5.0"    

  #  When using multiple platforms
  # s.ios.deployment_target = "5.0"
  # s.osx.deployment_target = "10.7"
  # s.watchos.deployment_target = "2.0"
  # s.tvos.deployment_target = "9.0"


  # ――― Source Location ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Specify the location from where the source should be retrieved.
  #  Supports git, hg, bzr, svn and HTTP.
  #

  #当前私有库资源的git地址，相应的tag，这里直接使用代码来标识使用和当前tag一样的值
  s.source       = { :git => "http://EXAMPLE/PrivateAdd_v2.git", :tag => "#{s.version}" }  


  # ――― Source Code ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  CocoaPods is smart about how it includes source code. For source files
  #  giving a folder will include any swift, h, m, mm, c & cpp files.
  #  For header files it will include any header in the folder.
  #  Not including the public_header_files will make all headers public.
  #

  # 需要引用的文件代码，classes文件夹下的所有.h.m文件。这里的路径都是相对路径
  s.source_files  = "Classes", "Classes/**/*.{h,m}" 
  # 排除掉哪些文件，不被包含的文件，这里是Classes/Exclude文件夹下的所有
  s.exclude_files = "Classes/Exclude"   

  # s.public_header_files = "Classes/**/*.h"


  # ――― Resources ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  A list of resources included with the Pod. These are copied into the
  #  target bundle with a build phase script. Anything else will be cleaned.
  #  You can preserve files from being cleaned, please don't preserve
  #  non-essential files like tests, examples and documentation.
  #

  # s.resource  = "icon.png"            # 资源文件
  # s.resources = "Resources/*.png"     # 所有资源文件

  # s.preserve_paths = "FilesToSave", "MoreFilesToSave"


  # ――― Project Linking ―――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  Link your library with frameworks, or libraries. Libraries do not include
  #  the lib prefix of their name.
  #

  # s.framework  = "UIKit"          # 依赖的系统framework，去掉后缀
  # s.frameworks = "SomeFramework", "AnotherFramework"

  # s.library   = "iconv"                   # 依赖的系统library
  # s.libraries = "iconv", "xml2", "z"      # 依赖的library，去除前缀和后缀，例如“libz.tbd”，则直接引用为"z"
  
  # s.vendored_frameworks = "*.framework", "*.framework"          #依赖那些静态库
  # s.vendored_libraries = "*.a", "*.a"           # 依赖哪些静态library


  # ――― Project Settings ――――――――――――――――――――――――――――――――――――――――――――――――――――――――― #
  #
  #  If your library depends on compiler flags you can set them in the xcconfig hash
  #  where they will only apply to your library. If you depend on other Podspecs
  #  you can include multiple dependencies to ensure it works.

  # s.requires_arc = true       # 是否开启ARC

  # s.xcconfig = { "GCC_PREPROCESSOR_DEFINITIONS" => 'HMTIMEINPREGNACYAPP=1', "HEADER_SEARCH_PATHS" => "$(SDKROOT)/usr/include/libxml2" }  
  # 这里配置一些内容，"GCC_PREPROCESSOR_DEFINITIONS"：定义一个宏，
  # "HEADER_SEARCH_PATHS"：如果引用该私有库，则会在主工程中寻找文件夹下的内容
  
  # s.dependency "JSONKit", "~> 1.4"        # 使用了pod，依赖哪些库

  # s.prefix_header_contents = <<-EOS     #这里会生成一个默认pch文件，作用于Xcode工程中添加的pch文件一样。
  #      #import <UIKit/UIKit.h>
  # EOS

end
```
这里有一个需要注意的地方，在创建私有库的时候，找不到MIT LICENSE证书，这里讲license修改一下

```
s.license      = { :type => "MIT", :file => "LICENSE" }
```

## 二级库subspec

```
  s.subspec 'UIView+Category' do |ss| 
    ss.source_files = "PrivateAdd_v3/PrivateAdd_v3/Classes/UIView+Category/*.{h,m}"
    ss.framework = "UIKit"
    # ss.vendored_libraries = '/*.a'
    # ss.vendored_frameworks = '/*.framework'
    # ss.resource = 'Moments/ForPregnacy/**/*.bundle', 'Moments/ForPregnacy/**/*.png', 'Moments/ForPregnacy/**/*.mp3'
  end
```
使用subspec关键字，并添加指定文件就行。

## 验证私有库
修改完相应的代码块之后，开始验证私有库配置是否正确。

### 本地验证

```
pod lib lint
// 如果使用了私有库，则使用--sources来标明，给出具体地址
pod lib lint --sources=[私有仓库repo地址],[私有库git地址],https://github.com/CocoaPods/Specs.git
```

如果出现错误，基本上都是podspec文件中的配置出现了问题，例如：

```
$ pod lib lint

 -> PrivateAdd_v2 (0.0.1)
    - WARN  | attributes: Missing required attribute `license`.
    - WARN  | license: Missing license type.
    - ERROR | description: The description is empty.
    - ERROR | [iOS] unknown: Encountered an unknown error (The `PrivateAdd_v2` pod failed to validate due to 1 error:
    - WARN  | attributes: Missing required attribute `license`.
    - WARN  | license: Missing license type.
    - ERROR | description: The description is empty.

) during validation.

[!] PrivateAdd_v2 did not pass validation, due to 2 errors and 2 warnings.
You can use the `--no-clean` option to inspect any issue.
```
这就需要将上面的错误修改掉。

一般情况下出现警告在后面添加`pod lib lint --allow-warnings`，就可以通过验证了。验证通过会提示passed validation。

如果依赖一些静态库（.framework，.a），则在命令后面添加`--use-libraries`，表示依赖了静态库。

如果报错不是很明显，这命令后添加`--verbose`，会输出验证的过程，并给出报错信息。

### 远端验证
`pod spec lint`，一般情况下私有库都是存在远程git上的，所以基本上只在第一次验证使用本地验证之后，就可以直接使用远端验证了。
但是一般情况下，创建的私有代码库如果是私有的（并不是官方的），这时候，我们在做远端验证时，需要添加代码地址。，不然pod会默认从官方repo查询。

```
pod spec lint --sources=[私有仓库repo地址],[私有库git地址],https://github.com/CocoaPods/Specs.git

// 或者只添加前面两项，最后一项可以忽略
pod spec lint --sources=[私有仓库repo地址],[私有库git地址]
```
如果出现一些警告，可以直接在命令最后添加`--allow-warnings`来屏蔽警告。例如：

```
pod spec lint --sources=https://gitee.com/devAlan/PrivateRepo,https://gitee.com/devAlan/PrivateAdd_v3.git,https://github.com/CocoaPods/Specs.git --allow-warnings

pod spec lint --sources=https://gitee.com/devAlan/PrivateRepo,https://gitee.com/devAlan/PrivateAdd_v3.git --allow-warnings
```

## 发布私有库
将podspec文件推送到私有空间repo，在发布时一定要记住，已经做了tag处理。

```
pod repo push [私有仓库repo] [私有库podspec文件地址] --allow-warnings
pod repo push [私有仓库repo] [私有库podspec文件地址] --sources=[私有仓库repo地址],[私有库地址],https://gitee.com/devAlan/PrivateAdd_v2.git --allow-warnings
```
由于基本上的所有命令操作都是在该私有库文件夹下，所以不用管podspec的文件位置。

```
pod repo push PrivateRepo PrivateAdd.podspec --allow-warnings
```

执行该命令时，相当于直接将podspec文件push到repo的Git地址上。

## 引用私有库

### 本地路径引用
做好验证之后，如果需要测试，可以直接在podfile中引用私有库的本地路径

```
// ../是回到上一级文件目录，是以当前podfile所在文件位置决定
pod 'Private_v2', :path => '../../Private_v2'
```
这个时候如果改动Private_v2中的代码就是直接改动。

### repo引用

在podfile中先添加source，然后直接pod引入，当然在引用私有库的时候，不仅需要需要私有仓库repo的权限，还需要有私有库的权限。

```
source 'https://gitee.com/devAlan/PrivateRepo'
 
pod 'Private_v2'

#或者使用下面的方式
#pod 'Private_v2', :git=> 'https://gitee.com/devAlan/PrivateRepo'
```


## pod 常用命令

1. 安装podfile中依赖的命令
`pod install` 
`pod update`
`--no-repo-update`   不需要更新repo
`--repo-update`   更新repo

2. repo相关
`pod repo list`
`pod repo remove xxx`   移除一个本地repo
`pod repo add [本地仓库repo名称] [远程repo地址]`  远程版本空间添加到本地

3. 私有库创建相关
`pod lib lint`      本地验证
`pod spec lint`     远程验证
`pod repo push [本地仓库repo] [*.podspec]`  发布版本到repo
`--source=[私有repo地址][私有代码库地址][pod地址]`
`--allow-warnings`  忽略warning
`--use-libraries`   使用了第三方的framework或者.a文件
`--verbose`         打印流程，可以快速定位失败原因

4. podspec中使用到的*
`/**` 子目录下所有文件夹
`/*`    目录下所有文件


## 遇到的问题

1. 私有库依赖主项目中的某些文件，如果是强制性的依赖，导致本地验证和远程验证都无法通过，所以更别说发布了。
    这种是没有完全的组件化。我们需要在podspec中添加HEADER_SEARCH_PATHS，添加指定位置文件路径。这里的路径也是相对路径。
    
    ```
    s.xcconfig = { "GCC_PREPROCESSOR_DEFINITIONS" => 'HMTIMEINPREGNACYAPP=1', 
    "HEADER_SEARCH_PATHS" => "$(SRCROOT)/usr/include/libxml2", $(SRCROOT)/../Commen/** }  
    ```
    
   但是这样做仍然无法通过验证，这就需要我们将私有空间repo pull到本地，手动创建版本号文件夹添加podspec文件，然后push。
   
2. 在上面的代码中还有一个 `GCC_PREPROCESSOR_DEFINITIONS`，这是定义一个宏，可以在Project-Build Settings - Preprocessor Macros下看到。
    
   可以配合一些没有必要引入私有代码库中的文件来使用。


