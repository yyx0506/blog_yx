---
title: "安装python3.11"
date: 2022-12-29
categories:
- tranquilpeak
- features
tags:
- python

# keywords:
# - python

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### 安装python3.11

#### 1:更换python 环境

```
之前一直使用的 是 python3.8 最近Python3.11 推出后看了一下官方介绍，确实改进了不少，那么就来玩一下
第一件事 直接去官网下载 https://www.python.org/downloads/release/python-3111/
下下来之后只需要一路下一步即可

一般来说 安装后的路径都在 /Library/Frameworks/Python.framework/Versions/ 下
我们在此路径下可以看到 有不同版本的 python 
下一步 删除原先的 python3 软连接 sudo rm /usr/local/bin/python3
将新的python3.11 的软连接指向 python3   sudo ln -s /usr/local/bin/python3.11 /usr/local/bin/python3

输入python3 --version，如果出现python的版本号即安装成功；
首先在终端里输入：which python3
export PATH=${PATH}:上面的路径
alias python="上面的路径"
alias pip3="上面的路径最后的python3 替换为pip3"
source ~/.bash_profile 

现在在 命令行输入python3. 就可以看到 现在的环境是Python3.11 了
```

### 2:pip3 永久换源

```
更换阿里 源
pip3 config set global.index-url https://mirrors.aliyun.com/pypi/simple/

阿里云:
https://mirrors.aliyun.com/pypi/simple/
中国科技大学:
https://pypi.mirrors.ustc.edu.cn/simple/
豆瓣:
https://pypi.douban.com/simple/
Python官方:
https://pypi.python.org/simple/
中国科学院:
https://pypi.mirrors.opencas.cn/simple/
清华大学:
https://pypi.tuna.tsinghua.edu.cn/simple/
```

### 3:使用 idea 运行python

```
idea 运营python 的好处就是 既可以写python 也可以写java 或者go 
下载破解idea 就各显神通了  我们打开一个项目 然后点击 file 如下图所示
```

![](/img/idea1.png)


然后 选择 sdk 即可  这个和 pycharm 完全不一样 毕竟是支持多种语言

![](/img/idea2.png)


#### 4: 安装python 包

```
在原先的项目上安装这个库
pip3 install pipreqs
然后在命令行 输入 pipreqs ./
可以将当前 项目所有用到的包全部导出来 

然后我们直接 pip3 install -r requirements.txt
即可安装成功所有包

```


#### 5：最后我们将此python 环境打包成镜像

```
我们创建一个dockerfile 文件 如下 选择使用python3.11 的镜像

FROM python:3.11.0-slim

WORKDIR /workplace/

RUN sed -i "s@http://deb.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list

RUN sed -i "s@http://security.debian.org@http://mirrors.aliyun.com@g" /etc/apt/sources.list

RUN apt-get clean && apt-get update && apt-get install gcc -y

COPY requirements.txt requirements.txt

RUN pip install --upgrade pip -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com && pip install -r requirements.txt -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com

```
最后执行 docker build 命令即可