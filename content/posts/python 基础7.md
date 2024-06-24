---
title: "python contextvars"
date: 2022-06-30
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - collections
# - python
# - calendar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### python 基础7

#### 1 ：contextvars

```
contextvars 是 python 3.7 以后新加的功能 名词解释就是 上下文 
这个模块提供了一组接口，可用于管理、储存、访问 局部上下文的状态。主要用于在异步环境中管理上下文变量。
先看一段代码
import asyncio
import contextvars

c = contextvars.ContextVar("只是一个标识, 用于调试")

async def get():
    # 获取值
    return c.get() + "===="

async def set_(val):
    # 设置值
    c.set(val)
    print(await get())

async def main():
    coro1 = set_("第一个携程")
    coro2 = set_("第二个携程")
    await asyncio.gather(coro1, coro2)

if __name__ == '__main__':
    asyncio.run(main())
----------------结果------------------
第一个携程====
第二个携程====
ContextVar 提供了两个方法，分别是 get 和 set，用于获取值和设置值 可以看到 两个携程的值是不会进行互相影响的，
也就是说数据是做到了在协程之间互相隔离的，再仔细看一下我们在 set_函数中设置的值 在get函数中获取到了值，当进行
await get() 相当于开启了一个新的协程，那么就意味着设置值和获取值不是在同一个协程当中，可是即便如此，我们依旧
可以获取到希望的记过，因为 Python 的协程是无栈协程，通过 await 可以实现级联调用。

 get 之前没有先 set，那么会抛出一个 LookupError
 c = contextvars.ContextVar("只是一个标识, 用于调试", default="这是一个默认值")
 除了在 ContextVar 中指定默认值之外，也可以在 get 中指定：
 print(c.get("这是一个默认值"))
 所以结论如下，如果在 c.set 之前使用 c.get：
 当 ContextVar 和 get 中都没有指定默认值，会抛出 LookupError；
 只要有一方设置了，那么会得到默认值；如果都设置了，那么以 get 为准；
 如果 c.get 之前执行了 c.set，那么无论 ContextVar 和 get 有没有指定默认值，获取到的都是 c.set 设置的值。
```

#### 2:contextvars token 

```
当我们调用 c.set 的时候，其实会返回一个 Token 对象：
import contextvars
c = contextvars.ContextVar("asyncio.Semaphore")
token = c.set("val")
print(token)
-------------------------结果--------------
<Token var=<ContextVar name='asyncio.Semaphore' at 0x7f7adfdb0c70> at 0x7f7adfe07080>
Token 对象有一个 var 属性，它是只读的，会返回指向此 token 的 ContextVar 对象。
Token 对象还有一个 old_value 属性，它会返回上一次 set 设置的值，如果是第一次 set，那么会返回一个 <Token.MISSING>。
利用这个功能可以对状态进行重置
除此之外 context 还有一个 run 方法
import contextvars

c1 = contextvars.ContextVar("context_var1")
c1.set("val1")
c2 = contextvars.ContextVar("context_var2")
c2.set("val2")
context = contextvars.copy_context()

def change(val1, val2):
    c1.set(val1)
    c2.set(val2)
    print('===',c1.get(), context[c1])
    print('===',c2.get(), context[c2])

context.run(change, "VAL1", "VAL2")
print('---',c1.get(), context[c1])
print('---',c2.get(), context[c2])
-------------------------结果--------------
=== VAL1 VAL1
=== VAL2 VAL2
--- val1 VAL1
--- val2 VAL2
我们看到 run 方法接收一个 callable，如果在里面修改了 ContextVar 实例设置的值，那么对于 ContextVar 而言只会在函数内部生效，一旦出了函数，那么还是原来的值。但是对于 Context 而言，它是会受到影响的，即便出了函数，也是新设置的值，因为它直接把内部的字典给修改了
```



#### tips：

```
以上就是 contextvars 模块的用法，在多个协程之间传递数据是非常方便的，并且也是并发安全的。如果你用过 Go 的话，你应该会发现和 Go 在 1.7 版本引入的 context 模块比较相似，当然 Go 的 context 模块功能要更强大一些，除了可以传递数据之外，对多个 goroutine 的级联管理也提供了非常清蒸的解决方案。
总之对于 contextvars 而言，它传递的数据应该是多个协程之间需要共享的数据，像 cookie, session, token 之类的，比如上游接收了一个 token，然后不断地向下透传。但是不要把本应该作为函数参数的数据，也通过 contextvars 来传递，这样就有点本末倒置了。
```

