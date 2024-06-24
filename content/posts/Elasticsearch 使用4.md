---
title: "Elasticsearch ik 分词器的安装 "
date: 2022-05-18
categories:
- tranquilpeak
- features
tags:
- Elasticsearch
# keywords:
# - Elasticsearch
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Elasticsearch 的使用4

这一篇来讲一下 ik分词器的使用 首先就是 https://github.com/medcl/elasticsearch-analysis-ik 直接到github 找到对应自己elasticsearch 版本的 ik 分词器 然后下载zip 版本  直接将此压缩文件 发送到对应的挂载plugins 的目录下面 创建一个 ik的文件

后直接解压 unzip 文件 重启 elastic search  后 再重启 kibana  

#### 下面基于kibana 来进行使用

```
ik 分词器 提供两种 ik_smart 和 ik_max_word 两种 分词算法 第一个为最少切分 第二个为最细粒度划分如下图所示
```
![](/img/esik1.png)

![](/img/esik2.png)


```
但是这样会遇到一个问题那就是当我们想要的词被拆开了怎么办如下图所示
```

![](/img/esik3.png)



```
ik 分词器可以自定义一下 该词添加到分词器的词典当中
cd /usr/local/docker/elasticsearch/plugins/ik/config  直接找到插件安装的目录下
然后 创建自己的字典: vim yyx.dic   将 耗子为汁 写上去
然后 vim IKAnalyzer.cfg.xml
最后直接 去重启es
我们再次执行这个操作
```
![](/img/esik4.png)

![](/img/esik5.png)



#### 后面甚至可以将收集到词统一放在dic内进行搜索  