---
title: iOS审核被拒-5.1.1 & 4.7.0 & 3.2.2
date: 2022-12-16 18:45:01
tags: 
    - iOS
    - 审核
categories:
    - app审核
---

# 5.1.1 Legal: Privacy - Data Collection and Storage & 4.7.0 Design: HTML5 Games, Bots, etc. & 3.2.2 Business: Other Business Model Issues - Unacceptable

被拒时间：2022.09.25 10:54
回复n次
过审时间：2022.10.13 23:13

值得一提的是，这个迭代是被拒的时间最长的一次，真的是搞心态啊，在此记录一下整体的过程。

## 第1次被拒
> Hello,
> 
> The issues we previously identified still need your attention.
> 
> If you have any questions, we are here to help. Reply to this message in App Store Connect and let us know. Bug Fix Submissions
> 
> The issues we've identified below are eligible to be resolved on your next update. If this submission includes bug fixes and you'd like to have it approved at this time, reply to this message and let us know. You do not need to resubmit your app for us to proceed.
> 
> Alternatively, if you'd like to resolve these issues now, please review the details, make the appropriate changes, and resubmit.
> 
> Guideline 5.1.1(v) - Data Collection and Storage
> 
> We noticed that your app supports account creation but does not appear to include an option to initiate account deletion.
> 
> Apps that support account creation must also offer account deletion to give App Store users more control of the data they've shared while using your app.
> 
> Next Steps
> 
> If your app already supports account deletion, reply to this message and let us know how to locate this feature. If your app does not support account deletion, revise your app to include an option to initiate account deletion.
> 
> If you are unable to offer account deletion or need to provide additional customer service flows to facilitate and confirm account deletion, either because your app operates in a highly-regulated industry or for some other reason, reply to this message in App Store Connect and provide additional information or documentation. If you have questions regarding your legal obligations, check with your legal counsel.
> 
> Keep these requirements in mind when updating your app to support account deletion:
> 
> - Only offering to temporarily deactivate or disable an account is insufficient.
> 
> - If users need to visit a website to finish deleting their account, include a link directly to the page on your website where they can complete the process.
> 
> - You may include confirmation steps to prevent users from accidentally deleting their account. However, only apps in highly-regulated industries may require users to use customer service resources, such as making a phone call or sending an email, to complete account deletion.
> 
> Resources
> 
> - Review frequently asked questions and learn more about the [account deletion requirements](https://developer.apple.com/support/offering-account-deletion-in-your-app).
> 
> - Apps that offer Sign in with Apple should use the [Sign in with Apple REST API](https://developer.apple.com/documentation/sign_in_with_apple/revoke_tokens) to revoke user tokens.
> 
> 拒绝原因：
> 5.1.1 Legal: Privacy - Data Collection and Storage

这个问题是苹果官方最新审核要求，在2022年6月30号以后，所有提交AppStore商城审核的应用程序支持帐户创建的 App 也必须提供帐户删除功能，否则将不会过审，拒绝原因就是Guideline 5.1.1(v) - Data Collection and Storage。

当看到这些信息的时候，我们以为是之前的原因，审核人员找不到注销的位置，所以我就把路径标注出来，并录屏作为附件进行了回复。结果，出乎意料，又被拒了，而且被拒的原因又多了一条。

## 第2次被拒

>Hello,
>
>The issues we previously identified still need your attention.
>
>If you have any questions, we are here to help. Reply to this message in App Store Connect and let us know. 
>
>Guideline 5.1.1(v) - Data Collection and Storage
>
>We noticed that your app supports account creation but does not appear to include an option to initiate account deletion.
>
>Apps that support account creation must also offer account deletion to give App Store users more control of the data they've shared while using your app.
>
>Next Steps
>
>If your app already supports account deletion, reply to this message and let us know how to locate this feature. If your app does not support account deletion, revise your app to include an option to initiate account deletion.
>
>If you are unable to offer account deletion or need to provide additional customer service flows to facilitate and confirm account deletion, either because your app operates in a highly-regulated industry or for some other reason, reply to this message in App Store Connect and provide additional information or documentation. If you have questions regarding your legal obligations, check with your legal counsel.
>
>Keep these requirements in mind when updating your app to support account deletion:
>
>- Only offering to temporarily deactivate or disable an account is insufficient.
>
>- If users need to visit a website to finish deleting their account, include a link directly to the page on your website where they can complete the process.
>
>- You may include confirmation steps to prevent users from accidentally deleting their account. However, only apps in highly-regulated industries may require users to use customer service resources, such as making a phone call or sending an email, to complete account deletion.
>
>Resources
>
>- Review frequently asked questions and learn more about the [account deletion requirements](https://developer.apple.com/support/offering-account-deletion-in-your-app).
>
>Guideline 4.7 - Design - HTML5 Games, Bots, etc.
>
>We noticed that your app includes code that is not embedded in the binary, but the software does not fulfill all of the requirements outlined in Guideline 4.7 of the App Store Review Guidelines or you have not provided us with an index of the software available in your app. Specifically, script message handlers in xxWebVC class were found not to be an appropriate implementation as it inappropriately expands standard web capabilities. 
>
>Next Steps
>
>To resolve this issue, please ensure the software offered in your app:
>
>– Is not offered in a store or store-like interface.
>
>– Is free or purchased using in-app purchase.
>
>– Does not allow e-commerce to transfer funds or game currency.
>
>– Only uses capabilities available in a standard WebKit view.
>
>– Is offered by developers that have joined the Apple Developer Program and signed the Apple Developer Program License Agreement.
>
>– Adheres to the terms of the App Store Review Guidelines.
>
>Additionally, if you have not done so already, it would be appropriate to provide an up-to-date index of the software and metadata available in your app in the Review Notes section of App Store Connect. This index should include the app name, developer name, game URL, and App Store Connect Team ID for the software.
>

### 翻译
此为机器翻译。

>你好，
>
>我们之前发现的问题仍然需要您的注意。
>
>如果您有任何疑问，我们随时为您提供帮助。 在 App Store Connect 中回复此消息并告知我们。
>
>准则 5.1.1(v) - 数据收集和存储
>
>我们注意到您的应用程序支持创建帐户，但似乎不包含启动帐户删除的选项。
>
>支持帐户创建的应用程序还必须提供帐户删除功能，以便 App Store 用户能够更好地控制他们在使用您的应用程序时共享的数据。
>
>下一步
>
>如果您的应用程序已经支持删除帐户，请回复此消息并告诉我们如何找到此功能。 如果您的应用不支持帐户删除，请修改您的应用以包含启动帐户删除的选项。
>
>如果您无法提供帐户删除或需要提供额外的客户服务流程来促进和确认帐户删除，无论是因为您的应用程序在高度监管的行业中运行还是出于其他原因，请在 App Store Connect 中回复此消息并提供 附加信息或文档。 如果您对您的法律义务有疑问，请咨询您的法律顾问。
>
>在更新您的应用程序以支持帐户删除时，请牢记这些要求：
>
>- 仅提供暂时停用或禁用帐户是不够的。
>
>- 如果用户需要访问一个网站来完成删除他们的帐户，请在您的网站上包含一个直接指向他们可以完成该过程的页面的链接。
>
>- 您可以包括确认步骤以防止用户意外删除他们的帐户。 但是，只有高度监管行业的应用程序可能需要用户使用客户服务资源，例如拨打电话或发送电子邮件来完成帐户删除。
>
>资源
>
>- 查看常见问题并了解有关帐户删除要求的更多信息。
>
>
>准则 4.7 - 设计 - HTML5 游戏、机器人等。
>
>我们注意到您的应用程序包含未嵌入二进制文件中的代码，但该软件未满足 App Store 审查指南第 4.7 条中概述的所有要求，或者您未向我们提供您的可用软件索引 应用程序。 具体来说，发现 xxWebVC 类中的脚本消息处理程序不是合适的实现，因为它不适当地扩展了标准的 Web 功能。
>
>下一步
>
>要解决此问题，请确保您的应用中提供的软件：
>
>– 不在商店或类似商店的界面中提供。
>
>– 免费或使用应用内购买购买。
>
>– 不允许电子商务转移资金或游戏货币。
>
>– 仅使用标准 WebKit 视图中可用的功能。
>
>– 由加入 Apple Developer Program 并签署 Apple Developer Program License Agreement 的开发者提供。
>
>– 遵守 App Store 审查指南的条款。
>
>此外，如果您还没有这样做，最好在 App Store Connect 的评论注释部分提供您的应用程序中可用的软件和元数据的最新索引。 该索引应包括软件的应用程序名称、开发者名称、游戏 URL 和 App Store Connect Team ID。
>

### 信息解读

#### Guideline 5.1.1(v) - Data Collection and Storage

这个还是关于account deletion，讲了一堆关于账号注销的。

苹果的新规严格要求App必须要设置一个功能，让用户可以“从App中删除他们的账户”，但并没有明确该功能具体要怎么设置。目前来看，这个“删除账户”功能可以很简单，也可以很复杂。有些开发者采取的方式是在个人中心设置注销功能，也有开发者引导用户到客服或发送一封电子邮件告知用户来彻底删除他们的信息，这个可以根据具体需求决定。通俗一点，就是我们须在app的个人中心的设置里面添加账号注销按钮，点击按钮弹窗提示用户是否确认账号注销操作，用户点击确认或取消执行对应操作即可过审。

所以在重新提交的包中，我们就把注销账号的功能移到了设置中，并明确注销功能是可用的。

#### Guideline 4.7 - Design - HTML5 Games, Bots, etc.

这里明确了在`xxWebVC`中，不适当的扩展了web功能，而且，应用程序中可能包含了未嵌入二进制文件中的代码。正因为我们忽略了未嵌入的二进制代码，造成了又一次的审核失败。

在`xxWebVC`中，代码的逻辑已经好久没有更新了，关于这个被拒信息，也没有找到合适的解决方案，而我们在这个迭代为了增加H5的秒开率，添加了web复用池的逻辑，所以大家一致认为，可能是这里的逻辑导致的。

然后我们注销掉了相关逻辑，重新打包上传，这里需要注意的是，即使重新打包上传，也不需要重新提审，只需要回复邮件说明情况，就可以继续进入审核队列。

## 第3次被拒

>Hello,
>
>The issues we previously identified still need your attention.
>
>
>If you have any questions, we are here to help. Reply to this message in App Store Connect and let us know. 
>
>Guideline 3.2.2 - Business - Other Business Model Issues - Unacceptable
>
>The primary purpose of your app is to encourage users to watch ads or perform marketing-oriented tasks, which is not appropriate for the App Store.
>
>Next Steps
>
>We encourage you to review your app concept and incorporate different content and features that are in compliance with the App Store Review Guidelines.
>
>Guideline 5.1.1(v) - Data Collection and Storage
>
>We noticed that your app supports account creation but does not appear to include an option to initiate account deletion.
>
>Apps that support account creation must also offer account deletion to give App Store users more control of the data they've shared while using your app.
>
>Next Steps
>
>
>If your app already supports account deletion, reply to this message and let us know how to locate this feature. If your app does not support account deletion, revise your app to include an option to initiate account deletion.
>
>If you are unable to offer account deletion or need to provide additional customer service flows to facilitate and confirm account deletion, either because your app operates in a highly-regulated industry or for some other reason, reply to this message in App Store Connect and provide additional information or documentation. If you have questions regarding your legal obligations, check with your legal counsel.
>
>Keep these requirements in mind when updating your app to support account deletion:
>
>
>- Only offering to temporarily deactivate or disable an account is insufficient.
>
>- If users need to visit a website to finish deleting their account, include a link directly to the page on your website where they can complete the process.
>
>- You may include confirmation steps to prevent users from accidentally deleting their account. However, only apps in highly-regulated industries may require users to use customer service resources, such as making a phone call or sending an email, to complete account deletion.
>
>Resources
>
>- Review frequently asked questions and learn more about the account deletion requirements.
>
>Guideline 4.7 - Design - HTML5 Games, Bots, etc.
>
>We noticed that your app includes code that is not embedded in the binary, but the software does not fulfill all of the requirements outlined in Guideline 4.7 of the App Store Review Guidelines or you have not provided us with an index of the software available in your app.
>
>Next Steps
>
>
>To resolve this issue, please ensure the software offered in your app:
>
>
>– Is not offered in a store or store-like interface.
>
>– Is free or purchased using in-app purchase.
>
>– Does not allow e-commerce to transfer funds or game currency.
>
>– Only uses capabilities available in a standard WebKit view.
>
>– Is offered by developers that have joined the Apple Developer Program and signed the Apple Developer Program License Agreement.
>
>– Adheres to the terms of the App Store Review Guidelines.
>
>Additionally, if you have not done so already, it would be appropriate to provide an up-to-date index of the software and metadata available in your app in the Review Notes section of App Store Connect. This index should include the app name, developer name, game URL, and App Store Connect Team ID for the software.
>
>拒绝原因：
>3.2.2 Business: Other Business Model Issues - Unacceptable
>4.7.0 Design: HTML5 Games, Bots, etc.
>5.1.1 Legal: Privacy - Data Collection and Storage

### 翻译

>您好，
>
>我们之前发现的问题仍然需要您的注意。
>
>
>如果您有任何疑问，我们随时为您提供帮助。 在 App Store Connect 中回复此消息并告知我们。
>
>指南 3.2.2 - 商业 - 其他商业模式问题 - 不可接受
>
>您的应用程序的主要目的是鼓励用户观看广告或执行面向营销的任务，这不适合 App Store。
>
>下一步
>
>我们鼓励您审查您的应用程序概念，并纳入符合 App Store 审查指南的不同内容和功能。
>
>指南 5.1.1(v) - 数据收集和存储
>
>我们注意到您的应用程序支持创建帐户，但似乎不包含启动帐户删除的选项。
>
>支持帐户创建的应用程序还必须提供帐户删除功能，以便 App Store 用户能够更好地控制他们在使用您的应用程序时共享的数据。
>
>下一步
>
>
>如果您的应用程序已经支持删除帐户，请回复此消息并告诉我们如何找到此功能。 如果您的应用不支持帐户删除，请修改您的应用以包含启动帐户删除的选项。
>
>如果您无法提供帐户删除或需要提供额外的客户服务流程来促进和确认帐户删除，无论是因为您的应用程序在高度监管的行业中运行还是出于其他原因，请在 App Store Connect 中回复此消息并 提供额外的信息或文件。 如果您对您的法律义务有疑问，请咨询您的法律顾问。
>
>更新您的应用程序以支持帐户删除时，请牢记这些要求：
>
>
>- 仅提供暂时停用或禁用帐户是不够的。
>
>- 如果用户需要访问一个网站来完成删除他们的帐户，请在您的网站上包含一个直接指向他们可以完成该过程的页面的链接。
>
>- 您可以包括确认步骤以防止用户意外删除他们的帐户。 但是，只有高度监管行业的应用程序可能需要用户使用客户服务资源，例如拨打电话或发送电子邮件来完成帐户删除。
>
>资源
>
>- 查看常见问题并了解有关帐户删除要求的更多信息。
>
>指南 4.7 - 设计 - HTML5 游戏、机器人等。
>
>我们注意到您的应用程序包含未嵌入二进制文件中的代码，但该软件不满足 App Store 审查指南第 4.7 条中概述的所有要求，或者您没有向我们提供可用软件的索引 你的应用程序。
>
>下一步
>
>
>要解决此问题，请确保您的应用程序中提供的软件：
>
>
>– 不在商店或类似商店的界面中提供。
>
>– 免费或使用应用内购买购买。
>
>– 不允许电子商务转移资金或游戏货币。
>
>– 仅使用标准 WebKit 视图中可用的功能。
>
>– 由已加入 Apple Developer Program 并签署 Apple Developer Program License Agreement 的开发人员提供。
>
>– 遵守 App Store 审查指南的条款。
>
>此外，如果您还没有这样做，最好在 App Store Connect 的评论注释部分提供您的应用程序中可用的软件和元数据的最新索引。 该索引应包括软件的应用程序名称、开发者名称、游戏 URL 和 App Store Connect Team ID。

### 信息解读

好嘛~~~  又多了一条被拒信息。这里不得不说，如果你的APP一直被拒，那他审查的程度就会越仔细。

### 3.2.2 Business: Other Business Model Issues - Unacceptable
关于这一条新增的被拒，其实已经给明了原因，就是鼓励用户观看广告或执行面向营销的任务。这个在不是游戏的APP中做这些是不被允许的。

关于另外2条被拒的信息，我们已经没有办法处理了，而且也没有给出确切的信息，所以我们打算通过电话沟通，所以回复了苹果审核我们需要更确切的信息。

# 回复信息

>尊敬的审核团队：
>	你们好！
>
>	我们已经收到了贵司的审核回复，但是我们感到很迷惑，不太明白审核团队所指的我们的应用存在的具体问题是什么。期间我们优化了代码，重新做了提交，但是还是被拒绝。
>
>	我们很乐意你们给出明确的原因，以便我们进一步修改我们的APP。
>
>	1、关于encourage users to watch ads or perform marketing-oriented tasks，我们会修改、优化代码之后重新打包提交审核。
>	2、关于account deletion，我们确认我们的APP有注销账号的功能（需要登录），路径为：我-->设置-->注销账号。我们需要更进一步的信息来了解具体的原因。
>	3、关于includes code that is not embedded in the binary，我们仔细检查了代码，并没有发现相关内容，还请指出具体原因。
>
>	综上，我们需要更进一步的信息，来帮助我们按照App Store 审核指南的真实意图修改代码，使我们的应用真正符合App Store 审核指南。
>
>
>	我们喜爱App Store这个平台，也希望维护这个平台的生态健康。使我们的app完全符合App Store 审核指南的要求，是我们工作的方向。
>
>	您可以电话沟通来帮助我们处理这次APP被拒绝。
>	电话：0086 187xxxx  姓名：xxx
>	我们将在在北京时间的9:00-21:00接听电话，我们需要中文服务。
>
>	期待您的回复！
>
>	最后，祝您工作愉快！

回复完成之后，就是等着就行，期间他们会回复邮件，确认电话沟通。

>Hello liu,
>
>Thank you for your response. Your call with an Apple representative is confirmed.
>
>An Apple Representative will call you on the number you provided in the App Review Information section of App Store Connect within the next 3 to 5 business days to discuss your app.
>
>We look forward to helping you address the issues we found in our review.
>
>Best regards,
>
>App Store Review

说的是3-5个工作日会电话联系，结果会很快就会有人联系，大概也就1天的时间。但是中间有一个插曲，导致我没有接到电话，然后他们会回复你没有人接通电话，并会给你一个电话号码让你在指定时间回拨过去。我一看好家伙，地址为加利福尼亚。

## 电话沟通

一定要提前准备好你要问的问题，一定要有调理的进行问答。如果可以，请进行录音，因为外国人说汉语还是会有点不太好理解的。

<b>问：关于4.7.0 Design: HTML5 Games, Bots, etc，应用程序包含未嵌入二进制文件中的代码，这一点我们很疑惑，能否详细说明？</b>

>答：APP中有一个H5小游戏，游戏中可以通过一些操作，浏览帖子，发帖、评论等操作可以获取积分，通过在系统WKWebView打开的H5页面，应该与Safari打开没有区别。
>这个游戏应该属于第三方软件。只要不是我们原生的代码，都属于第三方。而不是只要拥有账号权限的开发人员开发的。H5不应该调用客户端的API。
>

这里所说的大概意思就是，在APP中不应该存在H5游戏积分与客户端的交互兑换的逻辑。不应该存在h5调用客户端APP相关功能API来消耗积分。

<b>问：请问这个小游戏是哪个？能否给出具体的小游戏名称？</b>
>答：你们没有看到截图吗？（这个是他们的疏忽，确实没有给截图附件）。会补充邮件说明截图情况，并告知了我们使用的是哪个小游戏。

<b>问：H5有自己的一套积分系统，是否可以使用WKWebView？</b>
> 答：在APP中为什么要H5的游戏有一套积分系统，那要怎么消耗积分呢？

既然都已经电话沟通了，那就索性把其他的问题也问一下：

<b>问：关于3.2.2 Business: Other Business Model Issues - Unacceptable，这个能否详细说明哪里有鼓励用户观看广告吗？</b>
> 答：在H5小游戏中提供了观看视频可以获得积分的操作。

这个小游戏与上面问题中的小游戏是同一个。

<b>问：关于注销账号这个问题，我们确定我们有该功能，为什么每次都还会有这个？是还需要有什么特殊逻辑吗？</b>
> 审核答：你们现在注销账号的逻辑是什么？
> 我们：把注销账号功能相关逻辑说了一下，申请注销后，会发送验证码进行确认，需要用户14天不登录才会注销。
> 审核答：那就没有问题

最后结束通话，他们会发送一封邮件说明本次通话的大概内容，也就是通话的日志。所以这些未嵌入的二进制代码绝对不是在ipa包中的，那就是线上的的html代码。

我们把之前的代码回退之后，重新打包上传ipa。这里我们犯了一个错误，就是我们 没有真正理解整个上传的流程。

当重新上传ipa包之后，不要再次点提审，应该回复邮件说明情况。而我们不仅点了提审，还没有回复邮件说明，所以我们又被拒了。。。

## 第4次被拒

这次被拒跟第三次被拒的内容完全一致。所以我们回复了一下说明大概情况。

>尊敬的审核团队：
>	你们好！
>
>	我们已经收到了贵司的审核回复，在9月30日我们也通过电话进行了沟通，确定了被拒的具体原因，我们也修改并优化了代码，重新提交了审核，但还是被拒绝了。
>
>	1、关于Guideline 3.2.2 - Business - Other Business Model Issues - Unacceptable，通过电话沟通确定是与Guideline 4.7的拒绝内容是一致的。
>	2、关于Guideline 4.7 - Design - HTML5 Games, Bots, etc. 我们已经通过服务端进行了处理，下线了相关游戏。
>
>	我们认为的标准可能与App Store 审核指南要求的标准不一致，所以，我们需要更进一步的信息，来帮助我们按照App Store 审核指南的真实意图修改代码，使我们的应用真正符合App Store 审核指南。
>
>	我们喜爱App Store这个平台，也希望维护这个平台的生态健康。使我们的app完全符合App Store 审核指南的要求，是我们工作的方向。
>
>	最后，祝您工作愉快！


## 第5次被拒
又经历了漫长的等待之后，我们又被拒了。

>Hello,
>
>The issues we previously identified still need your attention.
>
>If you have any questions, we are here to help. Reply to this message in App Store Connect and let us know. 
>
>Guideline 2.1 - Information Needed
>
>We’re looking forward to completing the review of your app, but we need more information to continue.
>
>Next Steps
>
>Please provide detailed answers to the following questions in your reply to this message in App Store Connect:
>
>1. Does the game intended to display to end users in this version?
>
>拒绝原因：
>2.1.0 Performance: App Completeness

### 翻译
>你好，
>
>我们之前发现的问题仍然需要您的注意。
>
>如果您有任何疑问，我们随时为您提供帮助。 在 App Store Connect 中回复此消息并告知我们。
>
>准则 2.1 - 所需信息
>
>我们期待完成对您的应用程序的审核，但我们需要更多信息才能继续。
>
>
>下一步
>
>请在您在 App Store Connect 中回复此消息时提供以下问题的详细答案：
>
>1.游戏是否打算在这个版本中向最终用户展示？
>

这个就有点懵逼了，大家做APP的都知道，肯定是通过后台操作在审核的时候把某些模块隐藏了，而且大部分也都是这么干的，所以不要被发现。发现了就是欺骗，后果很严重（封号）。

面对灵魂拷问，不自信也要自信，并要坚定的回答审核人员。

### 回复
>尊敬的审核团队：
>	你们好！
>
>	我们已经收到了贵司的审核回复，基于之前的电话沟通，我们已经确定了被拒的具体原因，所以我们也修改并优化了代码，重新提交了审核。

>	关于 “Does the game intended to display to end users in this version? ”这个问题，我们已经做了严格的代码审查，并确定不会向用户展示相关游戏。
>
>	我们喜爱App Store这个平台，也希望维护这个平台的生态健康。使我们的app完全符合App Store 审核指南的要求，是我们工作的方向。
>
>	最后，祝您工作愉快！

到这里，也就是这一次审核被拒的整个流程。希望对大家有所帮助。
