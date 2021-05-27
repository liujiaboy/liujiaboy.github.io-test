---
title: 代码注入
date: 2021-05-25 13:55:24
tags:
    - 应用签名
categories:
    - 逆向
---


# 代码注入

我们按照上一章（应用重签名）的逻辑，先把程序跑起来。

然后再`.app`中显示包内容，查看可执行文件。这里我们使用`MachOView`工具进行分析，这里我们主要查看的是`Load Commands`。

![](MachOView.jpg)

`Load Commands`：加载命令。

在`Load Commands`里头，可以看到所有的Framework。点击每一个Framework可以看到这个Framework的执行路径。


## Framework注入

### 创建动态库

1. 在工程中选择`WeChat.xcodeproj`，然后在工程配置页面，选择左下角的加号”+“ -> ”iOS“ -> search ”Framework“。

    ![](add_framework.jpg)

2. 创建一个类，添加`+(void)load`方法，打印一串字符串。

    ```
    + (void)load {
        NSLog(@"\n\n hock success...\n\n");
    }
    ```


3. 重新运行工程（使用重签名）。这个时候我们的`NSLog`并不会执行，因为并没有链接到可执行文件。


    然后在项目工程`project` -> `show in finder` -> 找到对应的APP -> 显示包内容 -> Framework文件夹。可以看到我们添加的ALHook.framework。

    然后重新用`MachOView`工具打开可以执行文件（需要这个可执行文件），看看是否有链接ALHook.framework。这里是没有的。

### 链接动态库

通过`yololib`工具修改Mach-O文件，目的就是链接我们添加的动态库。

把`yololib`工具放到我们的工程文件中，与`appSign.sh`文件同级。然后把上面debug的可执行文件也同样复制过来。

然后执行下面的命令。

```
$ ./yololib [可执行文件] Frameworks/[添加的动态库].framework/[添加的动态库的可执行文件]

$ ./yololib WeChat Frameworks/ALHook.framework/ALHook
```

执行完命令之后，重新使用`MachOView`工具打开可执行文件。

![](MachO_ALHook.jpg)


### 重新压缩，打包ipa文件

1. 重新解压`8.0.2.ipa`文件。
2. 替换`Payload/xx.app -> 显示包内容`中的可执行文件，把上一部链接好的可执行文件做替换。
3. 重新压缩 `$ zip -ry WeChat.ipa Payload/`。
4. 把压缩后的ipa包重新放在工程目录APP文件夹下。

### 重新运行
    
这个时候重新运行，就可以看到我们的NSLog了。

![](hook_success.jpg)

## dylib注入

我们按照上一章（应用重签名）的逻辑，先把程序跑起来。

### 创建dylib

然后选择target -> "+" -> "macOs" -> 搜索"library" -> 选择"Library"，命名为"ALHook"。步奏与创建动态库类似。

### 修改ALHook

在`ALHook` -> "Build Setting"中配置

1. base sdk改为 `iOS`
2. code signing identify 改为 `iOS Developer`

然后再文件中添加load方法，注入代码输出一串文本。

### 添加依赖 Copy Files

在当前工程中拷贝dylib。

![](add_dylib.jpg)

这里需要注意的是，`run script`的顺序一定是在`Copy Files`上的，因为如果顺序反了，会导致最后才执行脚本。则把copy的dylib文件给重新覆盖掉，因为脚本文件执行的是app替换。

### 使用脚本执行

在上方动态库中我们是手动使用`/.yololib`工具进行链接的，这里我们使用脚本，在原来的`appSigh.sh`文件末尾添加一句代码：

```
# 链接手动添加的库，做代码注入
./yololib "$TARGET_APP_PATH/$APP_BINARY" "Frameworks/libALHook.dylib"
```

### 直接运行

直接运行代码，可以在Framework中看到`libALHook.dylib`文件。

运行成功之后，可以看到我们的输出。


# 代码注入流程总结

* Framework手动注入，是为了熟悉原理，真正操作的时候我们使用的都是脚本文件，从繁到简。
* Framework流程：
    1. Xcode新建Framework。
    2. 通过yololib工具是Mach-O文件链接Framework文件。
        * 所有的Framework加载都是由DYLD加载进入内存被执行的
        * 注入成功的库路径会写入到Mach-O文件的`LC_LOAD_DYLIB`字段中
* dylib注入流程：
    1. Xcode新建`dylib`库，然后修改"Build Setting"
        1. base sdk改为 `iOS`
        2. code signing identify 改为 `iOS Developer`
    2. 添加依赖，`Copy Files`将dylib文件拷贝到APP包中
    3. 通过yololib工具链接dylib文件。

这里需要注意的是，顺序不能错误，如下图：

![](script_framework.jpg)

# 真正的代码注入

我们使用Framework的形式进行注入。因为比较方便，dylib需要修改一些东西。

这里也直接使用dylib中使用脚本的方式进行注入。



