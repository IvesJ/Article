> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [mp.weixin.qq.com](https://mp.weixin.qq.com/s/oiKupyQU3FYy4Wuj0ByARw)

前言

如何解决传统 WebView 所面临的三大核心问题 —— 性能、外观和集成性，以及 Mobile Bridge 如何成为我们移动开发策略中的关键转折点，甚至帮助我们加速向 React Native 的迁移。今日前端早读课文章由 @  
Mauricio de Meirelles 分享，@飘飘编译。

译文从这开始～～

![](https://mmbiz.qpic.cn/sz_mmbiz_png/meG6Vo0MevjwmCyGkq6tsiaRPztfDFs3iaTFVuuZEY3O6sY9dSs4O1URYZGZU0by7XnQllJJ1Zml1z20BDgqHqWQ/640?wx_fmt=png&from=appmsg)

Shopify 的移动应用大约有 600 个界面，虽然这些界面都对商家使用移动端体验有所贡献，但并不是每个界面对日常操作都同样重要。

对于那些关键界面，我们毫无疑问地选择使用[原生](https://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651272426&idx=1&sn=dfb91e8f0ff8bb3bd61865a358578510&scene=21#wechat_redirect)或 React Native 来开发，以确保提供最好的使用体验。但如果将同样的开发方式应用到其他不那么关键的界面上，代价就非常高昂，同时还会严重拖慢开发进度。

我们的挑战很明确：我们需要一种高效的方式，把这些 “非关键” 界面集成到移动应用中，而不需要再为 Web 和移动端分别开发。

WebView 看起来是一个合理的选择，但它往往难以提供良好的用户体验 —— 经常显得慢、不流畅，而且跟原生界面风格不一致，容易让人感觉脱节。

我们不满足于这些限制，而是把它当作一次机会：我们是否能 “重塑” WebView，让它更快、更美观、看起来更像原生界面？这也正是我们创建 Mobile Bridge 框架的出发点。这个框架专门用来增强 WebView，让网页内容可以无缝融入我们的移动应用。

在这篇文章中，我们会分享我们是如何解决传统 WebView 在[性能](https://mp.weixin.qq.com/s?__biz=MjM5MTA1MjAxMQ==&mid=2651276269&idx=1&sn=013390327013622af87b6cec477e32bf&scene=21#wechat_redirect)、外观和集成方面的主要问题，以及 Mobile Bridge 如何成为我们移动开发策略中的关键工具，甚至帮助我们加速向 React Native 的迁移。

#### WebView 的难题

WebView 一直不被看好，主要是因为用户一眼就能看出它不是 App 的原生部分。它们看起来脱节、运行缓慢，给人不好的使用体验。

[视频详情](javascript:;)

#### Mobile Bridge：Shopify 的变革者

Mobile Bridge 所带来的改进是如此显著，以至于 WebView 现在已经成为我们移动端开发战略中不可或缺的一部分。我们大量使用 WebView 来实现重要但非关键的功能，从而避免在 Web 和移动端重复开发。

而对于应用中所有关键功能，我们仍然坚持使用原生或 React Native 来提供最优质的体验。

我们已经将 Mobile Bridge 开源为一个独立的库，并开始将其整合进 Shopify 的其他产品中，比如 Balance、POS 和 Shop。这意味着更多应用也能采用这种强大的混合开发模式，从而享受到更快的开发周期和更好的用户体验。

Mobile Bridge 彻底改变了我们对 Web 与移动集成方式的看法 —— 它让我们可以拥有 Web 开发的灵活性与速度，同时为用户提供接近原生的流畅体验。它还释放了大量工程资源，让我们能将更多精力投入到提升关键原生界面的质量上。

关于本文  
译者：@飘飘  
作者：@Mauricio de Meirelles  
原文：https://shopify.engineering/mobilebridge-native-webviews

![](http://mmbiz.qpic.cn/mmbiz_jpg/meG6Vo0MevgiaJGuXA28v4rRxoibyhrCjcAWn51S1CYXu0S95uXBUZTn6z15bA8ARkzAhdqgwxSBSl3lFCFAPbGg/640?wx_fmt=jpeg&wxfrom=5&wx_lazy=1&wx_co=1&tp=webp)

这期前端早读课  
对你有帮助，帮” 赞 “一下，  
期待下一期，帮” 在看” 一下 。