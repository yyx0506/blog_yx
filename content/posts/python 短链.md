
---
title: "python 短链"
date: 2022-07-03
categories:
- tranquilpeak
- features
tags:
- python
- 短链
# keywords:
# - collections
# - python
# - calendar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->
### python 短链

```
当我们对外提供一个分享链接时往往要携带上各种参数例如分享的是什么，是谁分享的等等其他参数就会导致这个链接很长，甚至于有的由于参数过多
不利于推广，所以就有了短链这个思想。
目前国内很多网站也会去提供相关的服务比如腾讯或者微博百度等，有一些只是提供简单的网址缩短服务，有一些则在网址缩短服务的基础上还增加了访客统计的功能，对一些推广链接来说，很有用。
```

短链的实现方法挺多的，一般来说就是长链接对应一个短链接 下面介绍几种实现方法

#### 1 redis 自增

```
此方法实现时建议使用lua脚本操作 因为redis 的弱事务性 所以选择Lua脚本话就是原子性操作
async def get_shortid(key):
    script = '''
    local key = 's:short:%s'
    local id = redis.call('get',key)
    if(id)
    then
        return id
    else
        local new_id= redis.call('incr','s:unionid')
        redis.call('set',key,new_id)
        redis.call('set',new_id,key)
        return new_id
    end''' % key
    redis = await AioRedisService.get()
    s = await redis.eval(script)
    return int(s)
得出的id 最好再进行一次加密防止被恶意用户撞库

```

#### 2:hashlib

```
import hashlib

from appdirs import unicode
from urllib3.connectionpool import xrange


def get_md5(s):
    s = s.encode('utf8') if isinstance(s, unicode) else s
    m = hashlib.md5()
    m.update(s)
    return m.hexdigest()


code_map = (
    'a', 'b', 'c', 'd', 'e', 'f', 'g', 'h',
    'i', 'j', 'k', 'l', 'm', 'n', 'o', 'p',
    'q', 'r', 's', 't', 'u', 'v', 'w', 'x',
    'y', 'z', '0', '1', '2', '3', '4', '5',
    '6', '7', '8', '9', 'A', 'B', 'C', 'D',
    'E', 'F', 'G', 'H', 'I', 'J', 'K', 'L',
    'M', 'N', 'O', 'P', 'Q', 'R', 'S', 'T',
    'U', 'V', 'W', 'X', 'Y', 'Z'
)


def get_hash_key(long_url):
    hkeys = []
    hex = get_md5(long_url)
    for i in xrange(0, 4):
        n = int(hex[i * 8:(i + 1) * 8], 16)
        v = []
        e = 0
        for j in xrange(0, 5):
            x = 0x0000003D & n
            e |= ((0x00000002 & n) >> 1) << j
            v.insert(0, code_map[x])
            n = n >> 6
        e |= n << 5
        v.insert(0, code_map[e & 0x0000003D])
        hkeys.append(''.join(v))
    return hkeys


if __name__ == '__main__':
    print(get_hash_key('http://www.baidu.com')  )
```

#### 3:进制转换

```

# 十进制转换到62进制
def encode62(id):
    num_list = []
    while (id > 0):
        remainder = id % 62
        num_list.insert(0, secret[remainder])
        id = id // 62
    return "".join(num_list).rjust(6, secret[0])


# 62进制转换到十进制
def decode62(secret_str):
    num_list = list(map(secret.index, secret_str))
    num_sum = 0
    for i, num in enumerate(num_list[::-1]):
        num_sum += num * 62**i
    return num_sum

print(encode62(138))
print(decode62(encode62(138)))

将的到数字和长链接一一对应存储到库里，然后使用进制转换将短链给到客户
```



#### tips :无论怎么做只需要短链和长链能够一一对应即可

