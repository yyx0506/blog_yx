
---
title: "Mysql 操作2"
date: 2022-01-18
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


### mysql 操作2

#### 1:建表

```
from sqlalchemy import Column, Integer, String, SmallInteger, \
    TIMESTAMP, text, MetaData, Table, BigInteger, JSON, Float

from common.config.mysql_config import *


metadata = MetaData()

GameVersion = Table('game_version', metadata,
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


def creae_table():
    # engine1 = await AioMysqlService.get('test')
    from sqlalchemy import create_engine
    engine = create_engine(f"mysql+pymysql://{PRO_USER}:{PRO_PASSWORD}@{PRO_HOST}:{PRO_PORT}/{PRO_DATABASE}?charset=utf8")
    metadata.create_all(engine)

if __name__ == '__main__':
    creae_table()
```

#### 2: 基本的orm 操作

```
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
  
 
```

