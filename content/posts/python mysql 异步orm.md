---
title: "python mysql 异步orm"
date: 2023-03-06
categories:
- tranquilpeak
- features
tags:
- mysql


thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### python mysql 异步orm

#### 1：首先准备环境配置

```
python3  使用的是 python3.11环境
aiomysql-core==0.0.9
aiomysql==0.1.1
官方文档：https://aiomysql.readthedocs.io/
github：https://github.com/aio-libs/aiomysql
```

#### 2:编写单例连接mysql

```
import aiomysql
from apps.app import AppCtx
from common.config.mysql_config import *
from aiomysql.sa import create_engine


class AioMysqlService:

    env = 'development'
    instances = {}
    engines = {}
    pools = {}

    configs = {
        'development': {
            'test': {'user': DEVE_USER, 'password': DEVE_PASSWORD, 'db': DEVE_DATABASE,
                         'host': DEVE_HOST, 'port': DEVE_PORT},
        },
        'production': {
            'test': {'user': PRO_USER, 'password': PRO_PASSWORD, 'db': PRO_DATABASE,
                         'host': PRO_HOST, 'port': PRO_PORT},
        }

    }

    @classmethod
    async def get_engine(cls, name, loop):  # 连接引擎
        '''获取engine
        '''
        engine = cls.engines.get(name)
        if engine is None:
            config = cls.configs[AppCtx.get_env()][name]  # 匹配对应的 mysql db
            cls.engines[name] = engine = await create_engine(loop=loop, autocommit=True, charset='utf8mb4', **config)
        return engine

    @classmethod
    async def get(cls, name, loop=None):
        '''获取engine
        '''
        instance = cls.instances.get(name)
        if instance is None:
            cls.instances[name] = instance = await cls.get_engine(name, loop)
        return instance

    @classmethod
    async def get_pool(cls, name, loop=None):
        '''获取连接池
        '''
        pool = cls.pools.get(name)
        if pool is None:
            config = cls.configs[AppCtx.get_env()][name]
            cls.pools[name] = pool = await aiomysql.create_pool(loop=loop, autocommit=True, charset='utf8mb4',
                                                                **config)
        return pool

    @classmethod
    async def close(cls):
        '''关闭
        '''
        instances = cls.instances
        for i in instances:
            if instances[i]:
                instances[i].close()
                await instances[i].wait_closed()
        cls.instances = {}
        cls.engines = {}
        pools = cls.pools
        for i in pools:
            if pools[i]:
                pools[i].close()
                await pools[i].wait_closed()
        cls.pools = {}

```

#### 2:创建一张表

```
import asyncio

from sqlalchemy import Column, Integer, String, SmallInteger, \
    TIMESTAMP, text, MetaData, Table, BigInteger, JSON, Float

from common.config.mysql_config import *

metadata = MetaData()

GameVersion = Table('game_version_test', metadata,
                    Column('id', Integer, primary_key=True, comment='id'),
                    Column('key', String(255), server_default="", comment='版号'),
                    Column('game_name', String(255),
                           server_default="", comment='游戏名字'),
                    Column('unit_name', String(255),
                           server_default="", comment='著作权人名称'),
                    Column('copyright_name', String(255),
                           server_default="", comment='出版单位名称'),
                    Column('game_type', String(255),
                           server_default="", comment='游戏内容类别'),
                    Column('operation_name', String(255),
                           server_default="", comment='运营单位'),
                    Column('approvalno', String(255),
                           server_default="", comment='批文号'),
                    Column('isbnno', String(255), server_default="", comment='版号11'),
                    Column('change_info', String(255),
                           server_default="", comment='变更信息'),
                    Column('info_id', Integer,
                           server_default="0", comment='补充id'),
                    Column('create_time', Integer,
                           server_default="0", comment='时间'),
                    Column('updated_at', TIMESTAMP, server_default=text(
                        "CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP")))


async def creae_table():

    from sqlalchemy import create_engine
    engine = create_engine(f"mysql+pymysql://{PRO_USER}:{PRO_PASSWORD}@{PRO_HOST}:{PRO_PORT}/{PRO_DATABASE}?charset=utf8")
    metadata.create_all(engine)
```

#### 3 基本操作

```
import time

from aiomysql_core import AioMysqlAlchemyCore
from sqlalchemy import select, between,desc
from common.models.game_models import GameVersion
from common.service.aiomysql_service import AioMysqlService


class Mysqltest1(object):


    @classmethod
    async def get_mysql_engine(cls):
        engine1 = await AioMysqlService.get('test')
        core = AioMysqlAlchemyCore(engine1)
        clause = select(
            [GameVersion.c.id, GameVersion.c.key, GameVersion.c.game_name, GameVersion.c.unit_name,
             GameVersion.c.copyright_name,
             GameVersion.c.game_type, GameVersion.c.approvalno, GameVersion.c.isbnno, GameVersion.c.operation_name,
             GameVersion.c.change_info, GameVersion.c.info_id, GameVersion.c.create_time]).where(
            between(GameVersion.c.id, 100, 200))
        rows = await core.query(clause)
        data = [dict(row) async for row in rows]
        print(data)

    @classmethod
    async def orm_operate(cls):
        engine1 = await AioMysqlService.get('test')
        core = AioMysqlAlchemyCore(engine1)
        p = GameVersion
        _id ,game_id = 1,1
        clause = p.update().where(p.c.id == _id).values(app_id=str(game_id), is_relation=1)
        await core.execute(clause)


    @classmethod
    async def get_did(cls):
        engine = await AioMysqlService.get('yx_gv')
        core = AioMysqlAlchemyCore(engine)
        clause = select([GameVersion.c.id]).order_by(
            desc(GameVersion.c.id)).where(GameVersion.c.id).limit(1)
        row = await core.get(clause)
        return row[0]

    #
    @classmethod
    async def update_test(cls):
        engine = await AioMysqlService.get('test')
        core = AioMysqlAlchemyCore(engine)
        update_dict = []
        time1 = time.time()
        for i in range(1,10000):
            dict_info = {
                 # 'id' : i,
                 'key': i+100,
             }
            # update_dict.append(dict_info)
        # print(update_dict)
        #     print(dict_info)
            clause = GameVersion.update().values(dict_info).where(GameVersion.c.id == i)
            await core.execute(clause)
        time2 = time.time()
        print(time2-time1)

    # def change_dict(self):
    #     dict_a = {
    #         'id':'名字'
    #     }
    #     print(json.dumps(dict_a))


if __name__ == '__main__':
    # asyncio.run(Mysqltest1().change_dict())
    Mysqltest1().change_dict()
```

