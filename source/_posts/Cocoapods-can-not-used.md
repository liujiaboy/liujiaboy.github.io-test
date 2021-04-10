---
title: Mac升级造成cocoapods不可用
date: 2020-03-10 17:00:52
tags:
    - Cocoapods
categories:
    - 杂记
---

Mac系统升级之后，造成pods不可用，这里找到了解决的方案，前提是已经安装了cocoapods。

```
sudo gem update --system
sudo gem install -n /usr/local/bin cocoapods -v1.5.3
pod setup
```

第一步：更新gem。
第二步：安装指定的cocoapods版本，我这里指定的是1.5.3
第三步：更新pod

