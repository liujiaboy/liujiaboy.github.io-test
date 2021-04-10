---
title: Universal Link
date: 2020-03-15 17:33:57
tags:
    - iOS
    - Universal Link

categories:
    - iOS
---


## 什么是Universal Links？
iOS9之后，Apple推出的一种通用链接，能够方便的通过https链接来启动APP，通过唯一的网址，不需要特别的schema就可以链接一个特定的视图到APP。
这也就设计到universal links的几个特性：

1. Unique：唯一性，不像自定义Url schemes那样，因为他使用到了你网站的http或者https链接
2. Secure：安全性，当用户安装应用时，iOS会检测你上传到服务器上的文件，以确保你的网站允许其代表应用打开你的应用。
3. Flexible：灵活性，当没有安装你的app时universal links也是可以正常使用的，当点击link时会直接在safari中打开url。
4. Simple：一个url可以为app和web提供服务。
5. Private：其他app可以与你的app通信，不需要知道你的应用程序是否安装。



## 配置Universal links
配置Universal links需要网站与app协同处理，两端都需要做一些工作。

### 创建并上传关联文件
首先，你的网站必须支持https。<br>
然后，创建apple-app-site-association文件，注意一定是没有.json后缀的，在未压缩的情况下，文件大小不能超过128KB。

```
{
    "applinks": {
        "apps": [],
        "details": [
            {
                "appID": "9JA89QQLNQ.com.apple.wwdc",
                "paths": [ "/wwdc/news/", "/videos/wwdc/2015/*"]
            },
            {
                "appID": "ABCD1234.com.apple.wwdc",
                "paths": [ "*" ]
            }
        ]
    }
}
```

<b>appID：</b>是由TeamID+BundleId组成，TeamID可以通过开发者账号来查看，Bundle ID可以直接在工程里查看。<br>

<b>paths：</b>是一个数组类型用来指定网站路径，并且是大小写敏感的：
使用* 指定整个网站，在域名下的任何地址都可以打开App。<br>
“/wwdc/news/”指定链接，以指定网站的某些部分.<br>
还可以使用“?”匹配任何单个字符，"/foo/*/bar/201?/mypage"

创建好apple-app-site-association文件后，将其上传到web服务器的跟目录下或者` .well-known `子目录下。<b>文件是通过https访问不需要有任何重定向。</b>然后你可以直接在浏览器中输入domain/apple-app-site-association或者domain/.well-known/apple-app-site-association访问你所上传的文件。

通过Apple的测试网站可以检测你上传的文件是否正确。[传送门](https://search.developer.apple.com/appsearch-validation-tool/)

<b>注意：</b>
> 在用Nginx处理文件的MIME Type配置时，在域名下针对该文件进行处理，也就是说response的Content-Type必须设置为”application/json“。
使用Nginx处理文件的MIME Type配置，在server的某个域名下针对该文件处理

```
location /apple-app-site-association {
    default_type application/json;
}
```


### 项目配置

1. 需要支持Universal links的话，需要将开发者中心的配置APPlication Services列表中的ASSociated Domains变成Enabled。

2. 在项目targets->Capabilities->Associated Domains中配置App link。在这里需要注意一下，有的域名为www.abc.com和abc.com，这两个对于Universal links来说是不同的域名，所以，如果你的网站是这两个都需要做处理的话，需要将这两个域名都放在associate domains列表下。在添加时都要将`applinks:`放在域名前面。
    例如：
    ```
    applinks:www.abc.com
    ```

3. 不需要手动的添加`entitlements`，当添加domain之后，系统会自动添加相应的文件到工程，如果没有的情况下，只能通过手动添加。 
4. 在AppDelegate中调用方法来处理Universal links的回调。

```objc
- (BOOL)application:(UIApplication *)application continueUserActivity:(NSUserActivity *)userActivity restorationHandler:(void (^)(NSArray * _Nullable))restorationHandler
{
	if ([userActivity.activityType isEqualToString:NSUserActivityTypeBrowsingWeb]) {
        NSURL *webUrl = userActivity.webpageURL;
        if ([webUrl.host isEqualToString:@"domain"]) {
            //打开对应页面
        }else{
            //不能打开，使用Safari 打开
            [[UIApplication sharedApplication]openURL:webUrl];
        }
    }
    return YES;
}
```

这里回传过来一个`NSUserActivity`类型的数据，对该数据处理来跳转相应的界面。也就不再多说了。
到此整体的流程就已经处理完了。相关的内容介绍还是推荐看[官方文档](https://developer.apple.com/library/content/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW2)，写的比较细致。

## 注意事项

1. apple-app-site-association文件上传一定不要有`.json`后缀，那些隐藏后缀名的需要注意。
2. web server必须支持https
3. 上传文件最好在根目录下，官方说也可以在根目录下的`.well-known`目录下。
4. Xcode中配置Associated Domians时，一定要添加`applinks:`前缀。最多添加20-30条。
5. app在<b>安装</b>的时候会访问`domain`获取`apple-app-site-association`文件,这个可以通过抓包来获取。不是在打开app时访问。
6. 直接dev打包到手机上应该就可以测试，保证网络畅通。
7. 最简单的方式检测Universal links是否有效，将那个链接拷贝的备忘录中（imessage、邮件），直接点击链接会跳转到App，或者长按，会在弹出的ActionSheet中第二个显示`在xxx中打开`。
8. 一定是从上级页面（网页）<b>点击触发</b> 的Universal links。
9. 用 Safari 打开目标域名，或者在其他 App 里用 SFSafariViewController, WKWebView, UIWebView 打开目标域名，都可以达到效果。

## 那些坑

### 1. 跨域跳转：[饿了么遇到的一个问题](http://mobilists.eleme.io/2016/01/10/%E7%AA%81%E7%A0%B4%E5%BE%AE%E4%BF%A1%E8%B7%B3%E8%BD%AC%E9%99%90%E5%88%B6%EF%BC%8DUniversal-Links%E9%82%A3%E4%BA%9B%E5%9D%91/)

这里的问题，就是我们将链接复制在备忘录里发现可以拉起App，但是放在网页里，点击却是没有任何效果，这个几乎就是跨域的锅。

这个应该是在iOS9.2之后出现的问题。假设当前网页是abc.com/a，在这里有一个链接跳转到abc.com/b,还是在同一个域名abc.com下。这时点击，系统将不会进行拉起App的操作，必须在不同的域名下比如abcd.com/b，这样才会根据关联文件去判断是否要拉起app。这个时候就需要在Xcode中添加一个域名在碰到跨域问题不能拉起App时，链接改为其他的域名。
[这里也有解释](https://stackoverflow.com/questions/32751225/ios9-universal-links-does-not-work/32751734#32751734)

### 2. 选择性跳转

这个也是一个坑，苹果会通过Universal links记录用户的行为习惯，当你通过一个Universal links成功的拉起了App，这时你发现在status bar右上角有一个小按钮（带一个小箭头）。当你点击了之后会成功的在Safari中打开，这时，苹果就认为你不需要这个Universal links拉起App，此后在通过这个Universal links点击时，都是在safari或者网页中打开，不会再拉起App。这个是有办法补救的，通过在Safari中点击链接长按，在app中打开。此后通过Universal links就可以拉起App了。也就说，苹果会记录最后一次Universal links的跳转情况来判断是否需要拉起app。

这里感觉第一个的跨域跳转与第二个的选择性跳转是很类似的，第一个在当前域名下点击Universal links不能拉起app就相当于当前系统认为你这个universal links本身就在浏览器中打开的，认为不需要拉起app。
> When a user taps a universal link that you handle, iOS also examines the user’s recent choices to determine whether to open your app or your website. For example, a user who has tapped a universal link to open your app can later choose to open your website in Safari by tapping a breadcrumb button in the status bar. After the user makes this choice, iOS continues to open your website in Safari until the user chooses to open your app by tapping OPEN in the Smart App Banner on the webpage.

### 相关网站

[官方文档](https://developer.apple.com/library/content/documentation/General/Conceptual/AppSearch/UniversalLinks.html#//apple_ref/doc/uid/TP40016308-CH12-SW2)

通过Apple的测试网站可以检测你上传的文件是否正确。[传送门](https://search.developer.apple.com/appsearch-validation-tool/)


