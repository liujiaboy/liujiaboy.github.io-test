---
title: iPhone越狱
date: 2021-04-11 11:30:03
tags:
    - 越狱
categories:
    - 逆向
---

# 1. 准备工作

1. 我们需要一台iPhone手机，根据自己的条件，要求系统是iOS11.0 - 14.3，写这篇博客时所能处理的越狱系统是这个，过期之后请自行查找资料。
2. 越狱的网站：[https://unc0ver.dev/](https://unc0ver.dev/)，下载安装包（ipa文件），可以按照对应的步奏进行操作，本篇所说的有另外一种，使用重签名的机制，比较简单。



# 2. 开始越狱

1. 打开我们的工程文件，请自行下载。
2. 在"Signing & Capabilities" 下，修改"Bundle Identifier"为自己的，修改"Team" 使用"personal team"。
3. 在"Build Phases" - "Run Script"下，先将script运行代码注释掉。
    
    ```
    // 重签名机制的脚本文件
    #./appSign.sh
    ```
4. 运行当前的空工程文件到手机上。
5. 运行完成之后，在回到"Build Phases" - "Run Script"下，将注释打开，重新运行。
6. "unc0ver"这个APP已经出现在手机上，点击APP，直接开始"Jailbreak"。手机重启几次之后，就可以了。
7. 在"Setting"中，可以选择自己想使用的选项。其中 "Restore RootFS"可以回到未越狱状态。

# 3. 为什么使用unc0ver

1. 它可以在越狱之后，通过简单的操作回到当初未越狱的状态，对系统侵扰是最低的。
2. 这个是一个安装在手机上的APP，可以随时删除。
3. 开发者更新比较及时。写本篇时，iOS系统可更新的版本是14.4.2，而unc0ver已经支持到14.3版本了。