
---
title: "sanic 介绍"
date: 2022-03-02
categories:
- tranquilpeak
- features
tags:
- sanic
- 博客
# keywords:
# - sanic
# - restful-api
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### 1:sanic 介绍 

```
python 的后端 框架 可能大家最熟知的就是 flask 和 django 但是 由于这两个框架出的比较早 受限于早期python 性能的影响所以
其 并发吞吐不是很优秀，在 async/await 出来后一个新的后端框架就诞生了，这就是sanic 
```

#### 2: 安装sanic 

```
pip install sanic
```

#### 3: 一个简单的server

```
from sanic import Sanic
from sanic.response import text

app = Sanic()


@app.route("/")
async def index(request):
  # 这里也可使用json
    return text('Hello World!')


if __name__ == "__main__":
    app.run(host="0.0.0.0", port=9999)
```

#### 4: sanic-restful-api

```
1 ---------------------下载相关包
pip3 install sanic-restful-api
2 ---------------------编写 server 

from sanic import Sanic
from sanic_restful_api import Api, Resource

app = Sanic()
api = Api(app)

class WelCome(Resource):
			async def get(self, request):
				return "WelCome Sanic-restful-api"
api.add_resource(WelCome, "/")

if __name__ == "__main__":
	app.run(host="0.0.0.0", post="9999")
	
3 ---------------------使用reqparse解析Api的参数

#reqparse.RequestParser():创建对象
#add_argument：接受参数 例如 参数名字为name 类型是str 
#parse_args(): 解析参数
# 
from sanic_restful_api import Resource, reqparse

parser_get = reqparse.RequestParser()

parser_get.add_argument("name", location="args", type=str, default="")

class WelCome(Resource):
			async def get(self, request):
          args = parser_get.parse_args(request)
          print(args)
          return args.name
```

#### 5 ： 