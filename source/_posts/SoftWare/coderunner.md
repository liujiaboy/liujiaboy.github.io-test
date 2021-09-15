---
title: coderunner使用问题
date: 2021-09-15 19:01:29
tags:
    - coderunner
---

CodeRunner本身是一个收费软件，功能强大，好处就不说了。在一些破解网站上可以找到破解版的，断网输入对应的激活码就能破解成功。

## 问题
每次重新打开，都需要重新破解。

## 解决方案
猜想的是每次打开CodeRunner时都会向其主站发送消息，查看是否激活，这里就是通过直接断开链接的方式来处理。

### step1
Mac系统下打开`/private/etc/`文件，hosts是文本。

### step2
在hosts文件中，增加一行：

```
# CodeRunner App
127.0.0.1 coderunnerapp.com
```

这样就可以验证成功了，不需要每次都重新激活。

