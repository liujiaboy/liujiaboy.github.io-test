---
title: iOS审核被拒梳理 - 2.1.0
date: 2022-12-14 18:31:19
tags: 
    - iOS
    - 审核
categories:
    - app审核
---

# 2.1.0 Performance: App Completeness

被拒时间：2022.11.17 07:50
回复时间：2022.11.17 11:16
过审时间：2022.11.18 07:46

> Guideline 2.1 - Information Needed
> 
> We're looking forward to completing our review, but we need more information to continue. Your app uses the AppTrackingTransparency framework, but we are unable to locate the App Tracking Transparency permission request when reviewed on iOS 15.5.
> 
> 
> Next Steps
> 
> Please explain where we can find the App Tracking Transparency permission request in your app. The request should appear before any data is collected that could be used to track the user.
> 
> If you've implemented App Tracking Transparency but the permission request is not appearing on devices running the latest OS, please review the available documentation and confirm App Tracking Transparency has been correctly implemented.
> 
> If your app does not track users, update your app privacy information in App Store Connect to not declare tracking. You must have the Account Holder or Admin role to update app privacy information.
> 
> Resources
> 
> - Tracking is linking data collected from your app with third-party data for advertising purposes, or sharing the collected data with a data broker. Learn more about tracking.
> - See Frequently Asked Questions about the [requirements for apps that track users](https://developer.apple.com/app-store/user-privacy-and-data-use/#permission-to-track).
> - Review developer documentation for [App Tracking Transparency](https://developer.apple.com/documentation/apptrackingtransparency).
> 
> 拒绝原因：
> 2.1.0 Performance: App Completeness

本次被拒主要是因为在某个机型上没有发现使用idfa权限申请的地方，但是确使用了idfa的信息。所以我们只要回复我们有idfa权限申请，没有区分版本就可以了。

另外需要补充的是，注意引用在info.plist文件中为什么要获取idfa信息。

# 回复信息

```
尊敬的审核团队：
	你们好！

	关于“unable to locate the App Tracking Transparency permission request when reviewed on iOS 15.5.”的问题，我们确定使用了相关API，但是我们不会对系统和设备进行区分，在代码中iOS14以上的设备都会请求该权限。之所以需要该权限，是因为我们的APP宝宝树孕育需要访问您的idfa信息，从而精准的推荐广告服务。

	安装APP之后，首次点击运行APP时会有获取该权限的弹窗，请查看附件中的视频和截图。

	我们喜爱App Store这个平台，也希望维护这个平台的生态健康。使我们的app完全符合App Store 审核指南的要求，是我们工作的方向。

	最后，祝您工作愉快！

消息附件：
idfa.MP4
idfa.PNG
```