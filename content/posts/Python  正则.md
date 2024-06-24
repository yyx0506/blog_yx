
---
title: "python 正则"
date: 2022-07-27
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - 正则
# - python

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Python  正则

#### 1 正则的作用

```
正则在任何语言都有对应的库 ，在我看来其实就是一种规范 是计算机科学的一个概念
接下来主要讲一下 正则在 python 的运用 

import re 
re 模块有很多的有用的函数 比如 compile match search findall finditer split sub subn 等函数 根据不同的需求使用不同的模块

比如最最常见的 匹配 邮箱 和匹配手机号码

def get_email(text):
    pattern = re.compile('\w[-\w.+]*@([A-Za-z0-9][-A-Za-z0-9]+\.)+[A-Za-z]{2,14}')
    email = pattern.search(text)
    if email:
        print(email[0])
    else:
        print('没找到邮箱')

def get_phone(text):
    pattern = re.compile("1[356789]\d{9}$")
    res = pattern.search(text)
    if res:
        print(res[0])
    else:
        print('没找到手机号码')

```

#### 2 python 实际项目中使用正则

```
一般来说爬虫清洗 很多时候都会用到 正则
比如  在一个项目中  我们获取到 数据中含有 标签 我们希望将所有的 标签全部去掉 
string = re.sub('\<.*?\>','',string)
或者 我们希望对 其中某个标签保留 其他的标签去掉

  def sub_replace(match_obj):
    tag = match_obj.group()
    if '<img' in tag:
    	return '\n'+tag
    else:
    	return re.sub('\<.*?\>','',tag)

regex = re.compile(r'(<?[^>]+>)')
result = regex.sub(cls.sub_replace, string)

又比如 我们从网页端拿到的数据 需要将在 script 中的 数据进行拿出来 （json  是嵌套在代码中的） 就需要添加一个标识符然后以};结尾即可
ss = re.findall(r'data\["tplData"]\s+=\s+(.*?});', res)
print(ss)


还比如 之前遇到的一些网站 是有 云锁 在请求端携带的 就需要一些正则的匹配去构建cookie 

    @classmethod
    def stringToHex(cls,string):
        length = len(string)
        hex_string = str()
        for i in range(length):
            hex_string += hex(ord(string[i]))[2:]
        return hex_string

    @classmethod
    def get_cookie(cls,url):
        hex_string = cls.stringToHex(url)
        cookie = {"srcurl": hex_string, "path": "/"}
        return cookie

    @classmethod
    async def get_yunsuo_red(cls,url):
        import requests
        s = requests.Session()
        r = s.get(url)
        url_2 = re.compile("self\.location\s*=\s*\"(.*?)\"").findall(r.text)[0]
        screen_date = "1920,1080"
        url_2 = url_2 + cls.stringToHex(screen_date)
        url_2 = urljoin(url, url_2)
        cookie = cls.get_cookie(url)
        s.cookies.update(cookie)
        r2 = s.get(url_2)
        url3 = re.compile("self\.location\s*=\s*\"(.*?)\"").findall(r2.text)[0]
        r3 = s.get(url3)
        r3.encoding = "gbk"
        return r3.text
```

