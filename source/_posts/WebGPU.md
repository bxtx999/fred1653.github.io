---
title: WebGPU —— Chrome Developers
date: 2022-05-25 11:49:23
categories: "WebGPU"
tags: ["WebGPU", "chrome"]
desc: chrome developers - webgpu 翻译
toc: true
---

> 本文是_(WebGPU - Chrome Developers)[https://developer.chrome.com/docs/web-platform/webgpu/]_的翻译。

WebGPU支持web上的高性能3D图形和数据并行计算。

<!-- more -->

## WebGPU是什么？

WebGPU是一个新的网络API，它暴露了现代计算机图形功能，特别是Direct3D 12、Metal和Vulkan，用于在图形处理单元（GPU）上执行渲染和计算操作。

![](https://wd.imgix.net/image/vvhSqZboQoZZN9wBvoXq72wzGAf1/WHoJmX2IU7roV4iabH6M.png?auto=format&w=845)

这个目标与WebGL系列的API相似，但WebGPU能够访问GPU的更多高级功能。WebGL主要用于绘制图像，但可以通过很大的努力重新用于其他类型的计算，而WebGPU为在GPU上进行一般的计算提供一流的支持。

在W3C的 "网络GPU "社区小组中，经过四年的开发，WebGPU现在已经准备好让开发者在Chrome中试用，并对API和着色语言提出反馈意见。

**"After a decade of WebGL bringing 3D graphics to the web and enabling all sorts of new experiences, it's now time to upgrade the stack and help web developers take full advantage of modern graphics cards. WebGPU arrives just in time!"**

> Mr.doob, Three.js 的开发者

**WebGPU gets us closer to the metal and it also unlocks the power of compute shader for Web developers. New 3D experiences can be built today on [Babylon.js Playground](https://playground.babylonjs.com/#XCNL7Y).**

> David Catuhe, Babylon.js 的开发者

{% raw %}
<video src='https://storage.googleapis.com/web-dev-uploads/video/vvhSqZboQoZZN9wBvoXq72wzGAf1/Xb7LvsJ5e8efTssp94c6.mov' type='video/quicktime' controls='controls'  width='100%' height='100%'>
</video>
{% endraw%}

> Babylon.js的demo：使用WebGPU的计算着色器功能模拟汹涌的大海

## 现状

| 步骤 | 状态 |
| --- | --- |
| 创建解释器 | [完成](https://gpuweb.github.io/gpuweb/explainer/)  |
| 创造规范的初始稿 | [正在进行中](https://gpuweb.github.io/gpuweb/) |
| 收集反馈并迭代设计 | [正在进行中](https://developer.chrome.com/docs/web-platform/webgpu/#feedback) |
| 初始试验 | [正在进行中](https://developer.chrome.com/origintrials/#/view_trial/118219490218475521) |
| 发布 | 未开始 |

## 如何使用 WebGPU

### 通过 about://flags 开启

要在本地试验WebGPU，而不需要原产地试用令牌，请在`about://flags`中启用`#enable-unsafe-webgpu`标志。

### 在初始试验阶段启用支持

从Chrome 94开始，WebGPU可在Chrome中作为原产地试用。原产地试用预计将在Chrome 105（2022年9月21日）结束。

原产地试验允许您尝试新功能，并就其可用性、实用性和有效性向网络标准社区提供反馈。欲了解更多信息，请参见[《网络开发者初始试验指南》](https://github.com/GoogleChrome/OriginTrials/blob/gh-pages/developer-guide.md)。要报名参加这项或其他初始试验，请访问[注册页面](https://developers.chrome.com/origintrials/#/trials/active)。

### 初始实验注册

1. 为你的初始试验[申请一个令牌](https://developer.chrome.com/docs/web-platform/webgpu/)。

2. 将令牌添加到你的页面。有两种方法可以做到这一点:

   - 在每个页面的头部添加一个 origin-trial <meta> 标签。例如，这可能看起来像。

     `<meta http-equiv="origin-trial" content="TOKEN_GOES_HERE">`

   - 如果你能配置你的服务器，你也可以使用`Origin-Trial` HTTP头来添加令牌。由此产生的响应头看起来应该是这样的：

     `Origin-Trial: TOKEN_GOES_HERE`

## 特征检测

检查是否支持WebGPU，使用：

```js
if ("gpu" in navigator) {
  // WebGPU is supported! 🎉
}
```

> 注意
>
> 由`navigator.gpu.requestAdapter()`返回的GPU适配器可能为`null`。

## 开始

WebGPU是一个低级别的API，就像WebGL一样。它非常强大，相当啰嗦，而且在深入研究之前需要了解关键的概念。这就是为什么我将在本文中链接到现有的高质量内容，以便你开始学习WebGPU。

- [WebGPU — All of the cores, none of the canvas](https://surma.dev/things/webgpu/)

- [Get started with GPU Compute on the web](https://developer.chrome.com/gpu-compute/)

- [A Taste of WebGPU in Firefox](https://hacks.mozilla.org/2020/04/experimental-webgpu-in-firefox/)

- [WebGPU for Metal Developers, Part One](https://metalbyexample.com/webgpu-part-one/)

- [Learn what key data structures and types are needed to draw in WebGPU](https://alain.xyz/blog/raw-webgpu)

- [WebGPU Explainer](https://gpuweb.github.io/gpuweb/explainer/)

- [WebGPU Best Practices](https://toji.github.io/webgpu-best-practices/)

## 浏览器支持

WebGPU在ChromeOS、macOS和Windows 10的Chrome 94中的特定设备上可用，未来将支持更多设备。通过在Chrome浏览器上运行
`--enable-features=Vulkan`，可以获得Linux实验性支持。对更多平台的支持也将随之而来。

完整的已知问题列表可在 [ot-caveats](https://hackmd.io/QcdsK_g7RVKRCIIBqgs5Hw) 中找到。

在撰写本文时，[Safari](https://webkit.org/blog/9528/webgpu-and-wsl-in-safari/)和[Firefox](https://hacks.mozilla.org/2020/04/experimental-webgpu-in-firefox/)的WebGPU支持正在进行中。

## 平台支持

就像在WebGL的世界里，一些库也实现了WebGPU。

- [Dawn](https://dawn.googlesource.com/dawn)是Chromium中使用的WebGPU的一个C++实现。它可以用来在C和C++应用程序中以WebGPU为目标，然后使用[Emscripten](https://emscripten.org/)移植到[WebAssembly](https://developer.mozilla.org/docs/WebAssembly)，并在浏览器中自动利用WebGPU的优势。

- [Wgpu](https://sotrh.github.io/learn-wgpu/#what-is-wgpu)是Firefox中使用的WebGPU的一个Rust实现。它被Rust生态系统中的各种GPU应用所使用，例如[Veloren](https://veloren.net/devblog-125/)，一个多人的体素RPG。

## 演示

- [WebGPU Samples](https://austin-eng.com/webgpu-samples/)

- [Metaballs rendered in WebGPU](https://toji.github.io/webgpu-metaballs/)

- [WebGPU Clustered Forward Shading](https://toji.github.io/webgpu-clustered-shading/)

## 安全与隐私

为了确保网页只能使用自己的数据，所有的命令在到达GPU之前都经过严格的验证。请查看WebGPU规范中的[恶意使用考虑部分](https://gpuweb.github.io/gpuweb/#malicious-use)，以了解更多有关驱动错误的安全权衡。

## 反馈

Chrome团队想要了解你使用WebGPU的经验。

## 告诉我们关于API的设计

是不是API或shading语言有什么地方没有像你预期的那样工作？或者是缺少实现你的想法所需的方法或属性？对安全模型有问题或意见？在相应的[gitHub-repo](https://github.com/gpuweb/gpuweb/issues/)上提交一个规格问题，或者将你的想法添加到现有问题中。

## 报告实现中出现的问题

您是否发现Chrome浏览器的实现存在错误？或者实现方式与规范不同？请在[new.crbug.com](https://new.crbug.com/)上提交一个错误。请确保包含尽可能多的细节，比如内部的`about:gpu`页面的内容、简单的重现说明，并在组件框中输入`Blink>WebGPU`。[glitch](https://glitch.com/)对于分享快速而简单的重现非常有用。

## 支持WebGPU

您是否计划使用WebGPU？您的公开支持有助于Chrome团队确定功能的优先次序，并向其他浏览器供应商展示支持这些功能的重要性。

使用标签[#WebGPU](https://twitter.com/search?q=%23WebGPU&src=recent_search_click&f=live)向[@ChromiumDev](https://twitter.com/ChromiumDev)发送一条推文，让我们知道您在哪里以及如何使用它。在StackOverflow上以[#webgpu](https://stackoverflow.com/questions/tagged/webgpu)为标签提问。

## 有用的链接

- [Public explainer](https://gpuweb.github.io/gpuweb/explainer/)

- [WebGPU API Spec](https://gpuweb.github.io/gpuweb/)

- [WebGPU Shading Language (WGSL)](https://gpuweb.github.io/gpuweb/wgsl/)

- [Chromium tracking bug](https://bugs.chromium.org/p/chromium/issues/detail?id=1156646)

- [ChromeStatus.com entry](https://chromestatus.com/feature/6213121689518080)

- Blink Component: [Blink>WebGPU](https://chromestatus.com/features#component%3ABlink%3EWebGPU)

- [TAG Review](https://github.com/w3ctag/design-reviews/issues/626)

- [Intent to Experiment](https://groups.google.com/a/chromium.org/g/blink-dev/c/K4_egTNAvTs/m/ApS804L_AQAJ)

- [WebGPU's matrix channel](https://matrix.to/#/#WebGPU:matrix.org)
