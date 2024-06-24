
---
title: "Mysql 操作1"
date: 2022-01-15
categories:
- tranquilpeak
- features
tags:
- mysql
- python
# keywords:
# - mysql

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### mysql 操作 1

#### 1:搭建mysql

```
语言我们选者的是 python 操作的方法是orm 
我们先搭建一个mysql

使用 docker-compose 进行搭建
version: '3'
services:
  mysql8:
    image: mysql:8.0
    container_name: mysql8
    restart: always
    environment:
      MYSQL_ROOT_PASSWORD: test_pwd
      TZ: Asia/Shanghai
    command:
      --character-set-server=utf8mb4
      --collation-server=utf8mb4_general_ci
      --default-authentication-plugin=mysql_native_password 
      --explicit_defaults_for_timestamp=true
      --lower_case_table_names=1
      --max_allowed_packet=128M;
    ports:
      - 3306:3306
    volumes:
      # 映射SQL文件
      - /srv/mysql/sql:/sql
      - /srv/mysql/data:/var/lib/mysql
      # ro只读模式
      - /etc/localtime:/etc/localtime:ro

```

#### 2:mysql 连接

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
            'yx_gv': {"user": DEVE_gv_USER, "password": DEVE_gv_PASSWORD, "db": DEVE_gv_DATABASE, "host": DEVE_gv_HOST,
                      "port": DEVE_gv_PORT},

        },
        'production': {
            'test': {'user': PRO_USER, 'password': PRO_PASSWORD, 'db': PRO_DATABASE,
                         'host': PRO_HOST, 'port': PRO_PORT},
            'yx_gv': {"user": DEVE_gv_USER, "password": DEVE_gv_PASSWORD, "db": DEVE_gv_DATABASE, "host": DEVE_gv_HOST,
                      "port": DEVE_gv_PORT},
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
还是一样我们采用单例模式
```

#### mysql 异步测试

```
import asyncio
from aiomysql_core import AioMysqlAlchemyCore
from sqlalchemy import select, between
from common.models.game_models import GameVersion
from common.service.aiomysql_service import AioMysqlService


class Mysqltest1(object):


    @classmethod
    async def get_mysql_engine(cls):
        engine1 = await AioMysqlService.get('yx_gv')
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

if __name__ == '__main__':
    asyncio.run(Mysqltest1().get_mysql_engine())
```







