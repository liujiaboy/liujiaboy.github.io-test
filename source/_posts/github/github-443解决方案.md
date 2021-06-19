---
title: github-443解决方案
date: 2021-06-19 11:12:52
tags:
    - github
categories:
    - github
---

# 出现的问题

在使用github的时候，执行`git pull`或者`git push`时，经常会出现以下错误：

> 【Failed to connect to github.com port 443: Operation timed out】

这个时候就一通百度、google发现有解决方案：

```
// 注意啊、这个是不行的
git config --global https.proxy http://127.0.0.1:1080
git config --global http.proxy http://127.0.0.1:1080
```

这个时候你可能觉得：OK终于解决了。

但是，可能再你下次使用的时候又会出现类似的问题，或者又有新的问题出现。

# 解决方案

## step 1

打开网站：[https://github.com.ipaddress.com/](https://github.com.ipaddress.com/)

![](step-1.jpg)

web页面不要关，一会要用

## step 2

打开网站：[https://fastly.net.ipaddress.com/github.global.ssl.fastly.net](https://fastly.net.ipaddress.com/github.global.ssl.fastly.net)

![](step-2.jpg)

web页面不要关，一会要用

## step 3

打开网站[https://github.com.ipaddress.com/assets-cdn.github.com](https://github.com.ipaddress.com/assets-cdn.github.com)

![](step-3.jpg)

web页面不要关，一会要用

## step 4

打开系统host，进行编辑，我这里使用的是Mac，命令如下：

```
sudo vim /etc/hosts
```

`sudo`命令需要输入密码，之后，把我们上面打开的3个web对应的ip和host绑定，如下图：

![](step-4.jpg)

```
# ip            对应的host    
# Github
140.82.114.4    github.com
199.232.69.194  github.global.ssl.fastly.net
185.199.108.153 assets-cdn.github.com
185.199.109.153 assets-cdn.github.com
185.199.110.153 assets-cdn.github.com
185.199.111.153 assets-cdn.github.com
```

ip以自己打开的那3个web显示的为准。Windows请自行百度如何操作host。

## step 5

如果设置了`http.proxy`和`https.proxy` http/https代理，需要取消代理。

```
git config --global --unset http.proxy
git config --global --unset https.proxy
```

## step 6

刷新DNS，如果机型不同，不起作用，请自行查看[还原OS X 中的DNS缓存](https://support.apple.com/zh-cn/HT202516)

```
https://support.apple.com/zh-cn/HT202516
```

到这里就可以正常使用了。


