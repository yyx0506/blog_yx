---
title: "python 闭包 装饰器"
date: 2022-06-23
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - 闭包
# - python
# - 装饰器
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python 基础2

#### 1:闭包

```
在函数内部再定义一个函数，内部函数用到了外边函数的变量，并且外部函数将内部函数的引用返回，那么将这个函数以
及用到的一些变量称之为闭包
```

#### 2:装饰器

```
装饰器的作用：在不改变原函数的情况下给函数增加功能！
装饰器由闭包和语法糖组成 装饰器使函数调用变慢了
def decorator(func):
    """this is decorator __doc__"""
    def wrapper(*args, **kwargs):
        """this is wrapper __doc__"""
        print("this is wrapper method")
        return func(*args, **kwargs)
    return wrapper

@decorator
def test():
    """this is test __doc__"""
    print("this is test method")

print("__name__: ", test.__name__)
print("__doc__:  ", test.__doc__)
---------------------------结果--------------
__name__:  wrapper
__doc__:   this is wrapper __doc__
我们发现test 的名字 以及描述属性都变成 wrapper 相当于 test 方法被 装饰过后 返回的就是 wrapper 的引用了，
也就是说test 方法指向了wrapper ,
为了不影响，Python的functools包中提供了一个叫wraps的decorator来消除这样的副作用。写一个decorator的时候，最好在实现之前加上functools的wrap，它能保留原有函数的名称和docstring

from functools import wraps
def decorator(func):
	"""this is decorator __doc__"""
	@wraps(func)
	def wrapper(*args, **kwargs):
		"""this is wrapper __doc__"""
		print("this is wrapper method")
		return func(*args, **kwargs)
	return wrapper

@decorator
def test():
	"""this is test __doc__"""
	print("this is test method")

print("__name__: ", test.__name__)
print("__doc__:  ", test.__doc__)
---------------------------结果--------------
__name__:  test
__doc__:   this is test __doc__
可以看出，wraps装饰器的作用就是保留被装饰对象的原始数据信息 消除(被装饰后的函数名等属性的改变)副作用。
```

#### 3: 装饰器实际运用

```
redis 缓存
def redis_cache(expire=60, is_replace=0):
    def decorator(func):
        @wraps(func)
        async def wrapper(*args, **kwargs):
            if not getattr(func, 'cache', True):
                return await func(*args, **kwargs)
            kws = ''.join(['%s%s' % (k, v)
                           for k, v in kwargs.items()]) if kwargs else ''
            ags = ''.join([str(arg) for arg in args] +
                          [func.__qualname__]) if args else ''
            key = 's:cache:%s' % (md_5('%s%s' % (ags, kws)))
            redis_cli = await AioRedisService.get('redis_n', 13)
            if not is_replace:
                s = await redis_cli.get(key)
                if s:
                    return msgpack.unpackb(s, raw=False)
            response = await func(*args, **kwargs)
            await redis_cli.set(key, msgpack.packb(response), expire=expire)
            return response
        return wrapper
    return decorator



@redis_cache(60)
async def get_xm_appinfos(app_id):
    keyword = f's:3:a:{app_id}'
    print(1)
    pika_cli = await AioRedisService.get('xm_pika_n', 0)
    print(2)
    result = await pika_cli.get(keyword)
    return result


async def get_infos():
    app_infos = await get_xm_appinfos(431355)
    print(app_infos)

if __name__ == '__main__':
    asyncio.run(get_infos())
    
    
当第一次执行时 没有缓存就会先读库  第二次在执行时有了缓存就会直接读缓存里的数据
此方法有两个参数 一个是缓存过期时间默认60秒
```

