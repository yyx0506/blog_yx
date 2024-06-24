---
title: "python 设计模式"
date: 2022-07-27
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - 单例
# - python
# - 工厂
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### python 设计模式

#### 1:设计模式的介绍

```
任何语言都有设计模式 这是一种思想  python 也不例外 如下

创建型模式：单例模式，工厂模式，建造者模式，原形模式
结构型模式：适配器模式，修饰器模式，外观模式，享元模式，模型-视图-控制器模式，代理模式
行为型模式：责任链模式，命令模式，解释器模式，观察者模式，状态模式，策略模式，模板模式
```

#### 2:单例模式

```
什么是单例模式？单例模式是指：保证一个类仅有一个实例，并提供一个访问它的全局访问点
单例模式是在创建对象时该类重写了__new方法
让他只执行一遍，它不仅仅是创建对象，
__new方法没创建一个对象执行一次，
他还是一个特殊的静态方法，
构建一个单例模式1： 饿汉式，现有对象，如果
要获取对象，直接将对象返回，以空间换时间
2：懒汉式：在获取对象时先判断是否有对象
有直接返回对象，没有创建一个对象返回
以时间换取空间的思想
例如我们使用数据库 的连接时 如果有多个地方的调用，我们可以使用 单例模式 保证只有一个调用
import aioredis
from apps.app import AppCtx
from common.config.redis import *


class AioRedisService():
    instances = collections.defaultdict(dict)
    redis_config_production = {
    }
    redis_config_development = {
        'ad_info': {"host": "", "port": "", "password": ""},
    }
        def __init__(self):
        pass

    @classmethod
    async def get(cls, redis_name='redis', db=0):
        if cls.instances.get(redis_name) is None:
            cls.instances[redis_name] = {}
        if cls.instances[redis_name].get(db) is None:
            config = cls.redis_config_production.get(
                redis_name) if AppCtx.get_env() == 'production' else cls.redis_config_development.get(redis_name)
            address = 'redis://%s:%s' % (config['host'], config['port'])
            loop = asyncio.get_event_loop()
            cls.instances[redis_name][db] = await aioredis.create_redis_pool(
                address=address,
                password=config['password'],
                db=db,
                loop=loop,
                encoding='utf-8'
            )
        return cls.instances[redis_name][db]

    @classmethod
    async def close(cls):
        '''关闭连接
        '''
        instances = cls.instances
        for i in instances:
            for k in instances[i]:
                if instances[i][k]:
                    instances[i][k].close()
                    await instances[i][k].wait_closed()
        cls.instances = collections.defaultdict(dict)
这就是单例模式的使用 只要你链接 的redis 数据库是同一个 无乱链接了多少个对象都只有一个实例 

```

#### 3: 工厂模式

```
工厂模式包涵一个超类。这个超类提供一个抽象化的接口来创建一个特定类型的对象，而不是决定哪个对象可以被创建。
为了实现此方法，需要创建一个工厂类并返回所需对象。 
当程序运行输入一个“类型”的时候，需要创建于此相应的对象。这就用到了工厂模式。在如此情形中，实现代码基于工厂模式，可以达到可扩展，可维护的代码。当增加一个新的类型，不在需要修改已存在的类，只增加能够产生新类型的子类。
"""
工厂模式：
根据需求产生对象
"""
#芭比、奥特曼、变形金刚、葫芦娃
class Toy:
    def __init__(self,name):
        self.name = name

    def feature(self):
        pass


class Bobby(Toy):
    def __init__(self,name):
        super(Bobby, self).__init__(name)

    def feature(self):
        print(self.name,"说：大家好，我是可爱的白雪公主")

class Ultraman(Toy):
    def __init__(self,name):
        super(Ultraman, self).__init__(name)

    def feature(self):
        print(self.name,("说：打怪兽打不过，飞回自己星球"))

class Transformers(Toy):
    def __init__(self,name):
        super(Transformers, self).__init__(name)

    def feature(self):
        print(self.name,"说：汽车人变形")

class HuluBrothers(Toy):
    def __init__(self,name):
        super(HuluBrothers, self).__init__(name)

    def feature(self):
        print(self.name,("说：妖精，还我师傅！"))

class Factory:

    @staticmethod
    def createToy(toyName):
        if toyName == "芭比":
            return Bobby("芭比")
        elif toyName == "奥特曼":
            return Ultraman("奥特曼")
        elif toyName == "变形金刚":
            return Transformers("变形金刚")
        elif toyName == "葫芦娃":
            return HuluBrothers("葫芦娃")

Factory.createToy("奥特曼").feature()

```

##### tips : 其余的模式我们后续在分享

