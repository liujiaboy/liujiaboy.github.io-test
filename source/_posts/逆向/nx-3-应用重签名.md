---
title: 应用重签名
date: 2021-05-21 15:42:58
tags:
    - 应用签名
categories:
    - 逆向
---



# 应用重签名

## codesign 命令

Xcode提供了签名工具`codesign`，我们可以通过几个命令iu可以完成重签名。

* `codesign -vv -d [WeChat.app]` 查看签名
* `$ security find-identity -v -p codesigning` 查看钥匙串力可以签名的证书
* `$ Codesign –fs [证书串] [filename]` 强制替换签名
* `$ Chmod +x [filename]` 给文件添加可执行权限权限
* `$ security cms -D -i ../embedded.mobileprovision` 查看描述文件
* `$ codesign -fs [证书串] --no-strict --entitlements=权限文件.plist APP包`
* `$ Zip –ry [file] [outfile] 将file压缩为outfile`
* `$ otool -l [WeChat]` 查看mach-O(可执行)文件， `cryptid`等于0说明是砸壳之后的（没有加密），等于1说明是加密的
* `$ otool -l WeChat > ~/Desktop/123.txt` 把文件导出到123.txt中


## codesign 签名步奏

首先我们需要拿到砸壳之后的`.ipa`文件，然后对这个文件解压缩，在`Payload`文件夹下看到对应的app文件。

### 查看签名

 `$ codesign -vv -d WeChat.app` 
 
 如果没有重签名，使用的证书还是微信自己的证书。
 
### 查看本地钥匙串中可用的证书
 
 `$ security find-identity -v -p codesigning`
 
 会列出钥匙串中可用的证书。

### 强制签名

选中`WeChat.app`，显示包内容

app包中主要有3个文件需要处理，我们需要对这3个文件做重签名：
1. 是资源文件的签名
2. Mach-O文件的签名
3. Framework的签名

但是内部的`PlugIns`文件夹中的插件，`watch`文件夹下的内容，使用免费的证书是无法签名的。可以删除掉。

然后我们先对Framework文件夹下的内容签名：

`$ codesign -fs "Apple Development: xxx" andromeda.framework`

所有的framework都要重签。

### 创建新的描述文件

1. Xcode新建工程，命名为`WeChat`,选择对framework重签名使用的证书。然后在手机上运行一下。
2. 在工程中选中`Products`,打开`WeChat.app`所在的文件夹。然后显示包内容，找到`embedded.mobileprovision`这个描述文件，复制到`WeChat.app`内。
3. 添加了描述文件之后，还需要修改`info.plist`的bundle id，因为描述文件中是有bundle id信息的。

### 更改权限
接下来我们看看`embedded.mobileprovision`的内容，因为我们需要拿到它的权限部分对`WeChat`重签名。

1. `$ security cms -D -i embedded.mobileprovision`查看`embedded.mobileprovision`的内容，找到`Entitlements`这个key对应的内容（注意拷贝时不需要key），复制到一个新的plist文件中，命名为`Entitlements`。
2. 然后把这个`Entitlements.plist`文件拷贝到`Payload`文件夹下，与`WeChat.app`同级。
3. `$ ls -l WeChat`查看Mach-O文件是否有可以执行权限，没有的话使用命令`$ chmod a+x WeChat`设置可执行权限。

### app重签

对APP重签，其实就是对内部Mach-O重签。
退回到`Payload`文件夹下，对`WeChat.app`签名：

`$ codesign -fs "Apple Development: xxx" --no-strict --entitlements=Entitlements.plist WeChat.app`

### 安装

使用Xcode - Window - Devices and Simulators，点击”+“，选择对应的app安装，安装完成之后，点击就能直接运行了。

> 注意：别登录，容易被封号，我们只是完成了万里长征的第一步而已。

这就是通过命令行使用`codesign`进行重签名的流程。

# Xcode重签名

我们对拿到的砸壳之后ipa文件进行解压缩，打开`Payload`文件夹：

1. 删除`PlugIns`、`Waatch`文件夹。
2. 使用命令行对Framework重签名。
3. 创建并运行一个空项目工程，项目名字需要与重签的`xx.app`文件名一致。并运行到手机上。
4. 修改xx.app内部的info.plist文件下的bundle id，与新工程一致。
5. 在工程下选择`Products`，打开`xx.app`所在的文件夹，把第4步修改后的app复制过来，直接替换`xx.app`。
6. 直接运行。

# codesign 与 Xcode签名比较
与`codesign`签名相比，直接使用Xcode重签名少了好几步：
1. 省略了添加描述文件
2. 省略了添加描述文件的权限
3. 省略了通过权限重签app

# Shell 重签

shell是一种特殊的交互式工具，它为用户提供了启动程序、管理文件系统中文件以及运行在系统上的进程的途径。
Shell一般是指命令行工具。它允许你输入文本命令，然后解释命令，并在内核中执行。
Shell脚本，也就是用各类命令预先放入到一个文本文件中，方便一次性执行的一个脚本文件。

> iTerm 切换bash和zsh的
> `$ chsh -s /bin/bash`
> `$ chsh -s /bin/zsh`

## 文件类型与权限

可以通过`$ls -l`命令查看权限

```
drwxr-xr-x  5  Alan  staff      160    10 19 2015  Public
[文件权限]  连接 所有者 所属组     文件大小  最后修改时间 文件名称
```

d：文件类型 表示目录。-表示 文件
前3位 rwx：文件所有者（user）的权限
中3位 r-x：这一组其他用户（group）的权限
后3为 r-x：非本组的用户（other）的权限，

| 缩写 | 释义 | 二进制 | 十进制 |
| :-: | ---- | ------| ------ |
|  r  | read 读|  0100|   4  |
|  w  | write 写| 0010|   2   |
|  x  |execut 可执行| 0001 | 1 |

我们经常会用的一个命令 `$chmod 755 filename`来改变file的权限

755表示的就是权限，分别为user、group、other的权限。
7： 4 + 2 + 1 = r + w + x
5： 4 + 1     = r + x

也可以使用符号类型来表示：

`$ chmod [u、g、o、a][+(加入)、-(除去)、=(设置)] [r、w、x] 文件名称`
u：user
g：group
0：other
a：all

例如 `$ chmod a+x 123.txt` 表示所有人都有可执行权限。

## shell重签名

就是在Xcode中，运行时执行shell脚本，自动帮我们处理签名的过程。在写shell脚本的时候，有以下几个步奏：

1. 解压ipa文件到Temp文件夹下
2. 将解压出来的app拷贝到工程下
3. 删除PlugIns和Watch相关的文件
4. 更新info.plist文件中bundle id
5. 给mach-o文件执行权限
6. 重签名framework
7. 替换签名

```
# ${SRCROOT} 它是工程文件所在的目录
TEMP_PATH="${SRCROOT}/Temp"
#资源文件夹，我们提前在工程目录下新建一个APP文件夹，里面放ipa包
ASSETS_PATH="${SRCROOT}/APP"
#目标ipa包路径
TARGET_IPA_PATH="${ASSETS_PATH}/*.ipa"
#清空Temp文件夹
rm -rf "${SRCROOT}/Temp"
mkdir -p "${SRCROOT}/Temp"

#----------------------------------------
# 1. 解压IPA到Temp下
unzip -oqq "$TARGET_IPA_PATH" -d "$TEMP_PATH"
# 拿到解压的临时的APP的路径
TEMP_APP_PATH=$(set -- "$TEMP_PATH/Payload/"*.app;echo "$1")
# echo "路径是:$TEMP_APP_PATH"

#----------------------------------------
# 2. 将解压出来的.app拷贝进入工程下
# BUILT_PRODUCTS_DIR 工程生成的APP包的路径
# TARGET_NAME target名称
TARGET_APP_PATH="$BUILT_PRODUCTS_DIR/$TARGET_NAME.app"
echo "app路径:$TARGET_APP_PATH"

rm -rf "$TARGET_APP_PATH"
mkdir -p "$TARGET_APP_PATH"
cp -rf "$TEMP_APP_PATH/" "$TARGET_APP_PATH"

#----------------------------------------
# 3. 删除extension和WatchAPP.个人证书没法签名Extention
rm -rf "$TARGET_APP_PATH/PlugIns"
rm -rf "$TARGET_APP_PATH/Watch"

#----------------------------------------
# 4. 更新info.plist文件 CFBundleIdentifier
#  设置:"Set : KEY Value" "目标文件路径"
# PlistBuddy工具修改info.plist文件中的BundleID为工程的BundleID
/usr/libexec/PlistBuddy -c "Set :CFBundleIdentifier $PRODUCT_BUNDLE_IDENTIFIER" "$TARGET_APP_PATH/Info.plist"

#----------------------------------------
# 5. 给MachO文件上执行权限
# 拿到MachO文件的路径
APP_BINARY=`plutil -convert xml1 -o - $TARGET_APP_PATH/Info.plist|grep -A1 Exec|tail -n1|cut -f2 -d\>|cut -f1 -d\<`
#上可执行权限
chmod +x "$TARGET_APP_PATH/$APP_BINARY"

#----------------------------------------
# 6. 重签名第三方 FrameWorks
TARGET_APP_FRAMEWORKS_PATH="$TARGET_APP_PATH/Frameworks"
if [ -d "$TARGET_APP_FRAMEWORKS_PATH" ];
then
for FRAMEWORK in "$TARGET_APP_FRAMEWORKS_PATH/"*
do

#签名 
#--force --sign 替换签名
# EXPANDED_CODE_SIGN_IDENTITY 当前工程的证书
/usr/bin/codesign --force --sign "$EXPANDED_CODE_SIGN_IDENTITY" "$FRAMEWORK"
done
fi
```

> 这里在第一次运行Xcode时，注意要先注释掉该shell脚本，运行一次之后再打开该shell脚本，重新运行。


# 总结

* codesign重签名
    1. 删除不能签名的文件：PlugIns(Extension)、Watch 
    2. 重签名Framework
    3. 给Mach-O可执行权限（`chmod`命令）
    4. 修改info.plist文件中的bundle id
    5. 创建新工程运行，拷贝描述文件（该描述文件要在iOS系统中信任过）
    6. 利用描述文件中的权限文件签名整个APP包
* Xcode重签名
    1. 删除不能签名的文件：PlugIns(Extension)、Watch 
    2. 重签名Framework
    3. 给Mach-O文件可执行权限（`chmod`命令）
    4. 替换掉Xcode运行后产生的APP包
* Shell重签名
    * shell切换
    * 老版本的Mac系统默认是bash，新系统（Sierra之后）默认shell是zsh
    * zsh的环境变量在`~/.zshrc`中，bash的环境变量在`~/.bash_profile`
    * 用户权限 chmod
        * 数字 r:4 w:2 x:1 `chmod 751 filename`
        * 字符 u:创建者 g：组 o：其他 a：所有 `chmod a+x filename`

还是推荐使用shell脚本重签名，因为以上两种方式虽然可行，但是会很麻烦，稍有不慎则会出现意外。
第二种使用Xcode签名，无法控制顺序，虽然做了app包替换，但是在运行时，还是会替换info.plist，导致有一些app安装之后，运行会出现问题。

