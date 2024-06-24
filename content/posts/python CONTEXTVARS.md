---
title: "python CONTEXTVARS"
date: 2023-04-13
categories:
- tranquilpeak
- features
tags:
- python


thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python **CONTEXTVARS**

#### 1 contextvars是什么

```
在 python 3.7 之后 新增了 模块contextvars 
这个 东西之前很多语言都有 现在 python 也终于有了  但是之前一直没有去研究这个东西，最近在异步编程中突然想到了这个 于是就研究一下

这个模块提供了一组接口，可用于管理、储存、访问 局部上下文的状态。
主要用于在异步环境中管理上下文变量。

contextvars 来源于 PEP-567 ，它提供了 APIs 允许开发者去保存，访问及管理 local context。 它与 thread-local storage (TLS) 类似。但它还支持维护异步代码的 context.

参数：
name： 必要位参； 用于检验和Debug.

default： 默参，且只能用keyword方式指定； 用于设定这个上下文变量的默认值。

属性：
name：只读特性。

get([default])：返回该上下文变量的值。未指定默认值且上下文变量无默认值时，抛出LookupError。

set(value)：设置上下文变量的值,返回一个与变量当前值相关的Token对象，可用于重置上下文变量的值到该次set之前。

reset(token)：使用token重置上下文变量的值。

```

#### 2 asyncio.Semaphore 与contextvars

```
例如  我们结合 asyncio.Semaphore 来做一个 爬虫的并发设计
假如你的并发达到2000个，程序会报错：ValueError: too many file descriptors in select()。报错的原因字面上看是 Python 调取的 select 对打开的文件有最大数量的限制，这个其实是操作系统的限制，linux打开文件的最大数默认是1024，windows默认是509，超过了这个值，程序就开始报错。所以我们用信号量去限制一下并发数量


from contextvars import ContextVar
import time,asyncio,aiohttp
sem_count = ContextVar("asyncio.Semaphore")
 
url = 'https://www.baidu.com/'
async def hello(url,semaphore):
    async with semaphore:
        async with aiohttp.ClientSession() as session:
            async with session.get(url) as response:
                return await response.read()
 
 
async def run():
    semaphore = sem_count.set(500) # 限制并发量为500
    to_get = [hello(url.format(),semaphore) for _ in range(1000)] #总共1000任务
    await asyncio.wait(to_get)
 
 
if __name__ == '__main__':

    loop = asyncio.get_event_loop()
    loop.run_until_complete(run())
    loop.close()
```

#### 3 contextvars 在python 异步中的体现

```
我们先看一个例子


import asyncio
from contextvars import ContextVar

var_test = ContextVar("test")

async def textss():
    try:
        return f"textss {var_test.get()}"
    except LookupError:
        return "textss? bad"


async def text():
    assert await textss() == "textss? bad"
    task_1 = asyncio.create_task(textss())
    coroutine_1 = textss()
    print(coroutine_1)
    var_test.set("world")
    task_2 = asyncio.create_task(textss())
    coroutine_2 = textss()
    print(coroutine_2)

    assert await task_1 == "textss? bad"
    assert await task_2 == "textss world"
    assert await coroutine_1 == "textss world"
    assert await coroutine_2 == "textss world"


asyncio.run(text())

结果为：
<coroutine object textss at 0x7ff2a027d9c0>
<coroutine object textss at 0x7ff2a029f7c0>

这个例子使用预先创造好的 ContextVar 对象去保存及访问当前 context 里的 test 变量。 这例子还暴露了一个问题，就是 coroutine 对象无法处理异步代码的 context 。 只有 asyncio.Task 在创建的时候会保存当前的 context，及自动的应用该 context 。


所以 如果你要创建一个 context，首先要做的是创建一些资源，保存在 context 中，然后在该 context 下执行异步代码，最后清理掉 context 中不需要的资源 。 最好的方式是使用 contextlib.asynccontextmanager 函数去做装饰器。

能够用 async-with 语法显得更符合 Python 哲学
资源初始化及清理代码处于同个函数内，更具有可读性

```

