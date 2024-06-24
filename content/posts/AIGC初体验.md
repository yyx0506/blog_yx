
---
title: "AIGC初体验"
date: 2023-07-01
categories:
- tranquilpeak
- features
tags:
- AIGC
# keywords:
# - AIGC

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### AIGC初体验

#### 1 aigc 是什么

```
AIGC全称AI-Generated Content，指基于人工智能通过已有数据寻找规律，并自动生成内容的生产方式。

AIGC既是一种内容分类方式，也是一种内容生产方式，还是一种用于内容自动生成的一类技术集合。

从创作者的角度看，内容生态的发展大致可以分成四个阶段：专家生成内容（PGC）、用户生成内容（UGC）、AI辅助生产内容 、AI生成内容（AIGC）。

2022年被称为AIGC元年。2021年之前，AIGC生成主要还是文字，而新一代的模型可以处理的模态大为丰富且支持跨模态产出，可以支持AI插画、文字生成配套视频等常见应用场景。



2022年，大量的AI绘画工具涌入市场，以Stable Diffusion为代表的软件纷纷开源，也大大降低了AI作图应用程序的开发门槛，进一步将AI概念推向新的高潮。



在主流的短视频平台、社交平台，逐渐充斥着大量的AI绘画内容，微博的相关话题总阅读数2.2亿，抖音平台中“AI绘画”特效使用近2600万人。
```



#### 2:安装步骤

```
市面上目前主流的是 Stable Diffusion WebUI 搭配模型去生成的 现在我们来尝试的去做一下工作


第一步:下载git
https://git-scm.com/download/win
第二步:下载python  注意必须是3.10之后的版本 建议下载最新的
https://www.pvthon.org/downloads/release/python-3106/
第三步: 下载stable diffusion
https://github.com/AUTOMATIC1111/stable-diffusion-webui
如何打开cmd控制台。打开想要安装的目录，选中文件夹路径 (在上面)，输入cmd，回车。
第四步: 下载model
https://huggingface.co/runwayml/stable-diffusion-v1-5
r载完mode后把mode解压，粘贴到 stable dfusion-webui/modelstable dfsion/底下，我这里下的mode是基mode，以较小，也可以专门下一些适合自己的mode下载依赖时，比如gfpgan等会遇到网络问题，要改launch文件，在原来的github网址前面加上镜像网址，这样就不会遇到网络问题了.
还可以去 https://civitai.com/ 下载需要的MODEL 

```

#### 3: 安装CUDA核心

```
https://developer.nvidia.com/cuda-toolkit-archive

先进入到CMD 中输入 nvidia-smi 查询自己的显卡对应的版本号 随后进入尚需链接进行下载

目前主要用到的是 文生图 和 图生图

下一步 就可以 输入 文字 来生成你想要的到图了 （但是他目前只能识别英文，重点是你要知道他的提示词的作用）
例如  一个少年在车水马龙的街上奔跑，正下着大雨，电闪雷鸣 
A young man was running on the busy street, with heavy rain and thunder and lightning
我们将翻译过来的词语放到TEXT内部就可以生成图了，非常的方便


```

