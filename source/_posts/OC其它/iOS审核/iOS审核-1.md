---
title: iOS审核被拒梳理 - 1.4.1 & 2.3.0
date: 2022-12-12 15:13:19
tags: 
    - iOS
    - 审核
categories:
    - app审核
---

# 1.4.1 Safety: Physical Harm & 2.3.0 Performance: Accurate Metadata

被拒时间：2022.12.09 03:21
回复时间：2022.12.09 15:20
过审时间：2022.12.10 12:16

```
Bug Fix Submissions
 
The issues we've identified below are eligible to be resolved on your next update. If this submission includes bug fixes and you'd like to have it approved at this time, reply to this message and let us know. You do not need to resubmit your app for us to proceed.

Alternatively, if you'd like to resolve these issues now, please review the details, make the appropriate changes, and resubmit.


Guideline 1.4.1 - Safety - Physical Harm

Your app provides medical diagnoses or treatment advice but does not include the required medical disclaimer.

Next Steps

Please revise your app's description to include a disclaimer reminding users to seek a doctor’s advice in addition to using this app and before making any medical decisions.

Once you have made the appropriate changes to your app’s description, please reply to this message in App Store Connect, and we will continue with the review.

Guideline 2.3 - Performance - Accurate Metadata

We were unable to locate some of the features described in your screenshot.

Next Steps

If these features are located in your app, please reply to this message in App Store Connect to provide information on how to locate them.

Alternatively, please revise your app to ensure that these features are fully implemented or revise your app description, release notes, and screenshots to remove all references to the features.

Please see attached screenshot for details. 
```

出现这种问题，如果已经提前说明了<b>Bug Fix Submissions</b>，那就好说了，我们可以直接回复有相应的bug fixes，为了不影响用户体验，所以需要尽快审核，关于相关内容的修改，会在下次提交更新时做出修改。一般情况下，回复邮件之后，等24小时左右就会过审。但是在下次提交时，一定要把对应的问题解决掉。

如果想解决对应的问题，就需要继续梳理问题原因了。

## Guideline 1.4.1 - Safety - Physical Harm
```
Your app provides medical diagnoses or treatment advice but does not include the required medical disclaimer.
```

翻译：APP提过了医疗诊断或治疗建议，但不包含所需的医疗免责声明。

### 解决
在APP信息页面，描述中增加免责声明，可以参考一下“丁香园”APP，为了安全保险起见，在用户协议或者相关协议中，添加对应的免责声明。

## Guideline 2.3 - Performance - Accurate Metadata

```
We were unable to locate some of the features described in your screenshot.
```

翻译：无法找到屏幕截图中描述的某些功能。

### 问题描述

这种情况属于，我们添加的APP截图（iOS预览和截屏）中提供了某些功能，但是审核人员却没有找到对应的路径。

通常情况下，他们会提供对应的截图，我们只需要根据截图内容，找到对应的实现路径就可以。
    
### 解决
这个可能会出现两种情况：

1、查看截图内容，是否为我们已经下线的功能。

我们犯的错误是，APP提审是另外一个团队，在其中一个版本更新预览图片时，有一些尺寸没有删除干净，4.7英才显示屏下对应的图片还是老的功能，但是已经下线了。所以找不到对应的路径。

这种情况，就需要删除对应的预览截图即可。回复邮件时说明情况。

2、如果确实存在该功能，但是审核人员没有找到。

这种情况，我们只需要回复邮件给出明确路径，并添加录屏附件即可。


# 最后回复

```
尊敬的审核团队：
	你们好！

	我们这个版本的更新包含了一些bug fixes，我们希望尽快得到审核，以免影响用户体验。
	关于提到的两个问题，我们会在下一次更新中解决。

	我们喜爱App Store这个平台，也希望维护这个平台的生态健康。使我们的app完全符合App Store 审核指南的要求，是我们工作的方向。

	最后，祝您工作愉快！
```

    



