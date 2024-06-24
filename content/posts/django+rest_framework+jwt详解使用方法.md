---
title: "Django+Rest_framework+JWT详解使用方法"
date: 2019-05-28
categories:
- tranquilpeak
- features
tags:
- Django
- JWT
# keywords:
# - disqus
# - google
# - gravatar
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

## django rest_frameworker jwt

- 首先需要声明，rest_frameworker jwt 是基于django自带的认证系统来实现的

  (也就是说我们的用户表(user)直接继承django自带的AbstractUser表，在此基础上添加字段)

- rest_frameworker jwt token的生成

  ```python
  from rest_framework_jwt.settings import api_settings
  
  class lll(APIView):
      def get(self,request):
          user = User.objects.get(id=1)
          #token生成过程
          jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
          jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
          #往添加token的数据
          payload = jwt_payload_handler(user) #这里需要修改为自己的数据
          #生成对token进行加密
          token = jwt_encode_handler(payload)
  
          return Response({'code':200,'token':token})
  ```

---

  自己定义注册用户，返回自己要的响应

  ```python
  #views.py
  #返回的响应 可以自己添加
  class Reg(APIView):
      #用户注册
      def post(self,request):
          #往用户表里添加用户 并且返回用户信息和token
          User.objects.create_user(username='khgiuh',phone='123456',password="123456") 
          
          user = 查找当前用户信息
          jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
          jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
          #往添加token的数据
          payload = jwt_payload_handler(user) #这里需要修改为自己的数据
          #生成对token进行加密
          token = jwt_encode_handler(payload)
          
          return Response({
                  'code':200,
               'token':token
              })
      
      #用户登录只返回 code：200，不会token(因为没用obtain_jwt_token)
      def get(self,request):
          #用来验证登录用户信息  参数只能是用户名 和密码   true：返回用户名  flase：返回None
          r = authenticate(username="peter",password="123456")
          user = 查找当前用户信息
          jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
          jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
          #往添加token的数据
          payload = jwt_payload_handler(user) #这里需要修改为自己的数据
          #生成对token进行加密
          token = jwt_encode_handler(payload)
          
          
          print(r)
          return Response({
                  'code':200,
               'token':token
              })
  ```

  ```python
  #models
  from django.contrib.auth.models import AbstractUser
  
  class User(AbstractUser):
      #创建自己需要的字段
      username = models.CharField(max_length=32,unique=True) #unique  唯一认证
      phone = models.CharField(max_length=32,unique=True)
  
      class Meta:
          db_table = 'user'
  ```

- 在settings里导入jwt配置

  ```pytohn
  AUTH_USER_MODEL='project_cs.User'   # 指定使用users APP中的 model
  (#自己定义的user表继承了django自带user表，通过上面的赋值来调用自己定义的user表)
  ```

  ---

## 通过序列化器注册用户并将用户信息和token 返回给前端

- 创建自己的用户表(user)并继承django的AbstractUser

  ```python
  #models
  from django.contrib.auth.models import AbstractUser
  
  class User(AbstractUser):
      #创建自己需要的字段
      username = models.CharField(max_length=32,unique=True) #unique  唯一认证
      phone = models.CharField(max_length=32,unique=True)
  
      class Meta:
          db_table = 'user'
  ```

- 在settings里导入jwt配置

  ```pytohn
  AUTH_USER_MODEL='project_cs.User'   # 指定使用users APP中的 model
  (#自己定义的user表继承了django自带user表，通过上面的赋值来调用自己定义的user表)
  ```

- 视图

  ```python
  from rest_framework.views import APIView
  from rest_framework.response import Response
  from rest_framework.request import Request
  from project_cs.models import User
  
  #用来验证用户
  from django.contrib.auth import authenticate
  
  
      
      
  #通过序列化器注册用户并将用户信息和token 返回给前端
  # 用户注册
  class RegisterView(APIView):
      def post(self, request, *args, **kwargs):
          serializer = UserSerializer(data=request.data)
          if serializer.is_valid():
              serializer.save()
              return Response(serializer.data, status=201)
          return Response(serializer.errors, status=400)
  
  
      
  # 重新用户登录返回函数 (登录url调用obtain_jwt_token方法，可以使用下面的返回函数)
  def jwt_response_payload_handler(token, user=None, request=None):
      '''
      :param token: jwt生成的token值
      :param user: User对象
      :param request: 请求
      '''
      #指定返回信息
      return {
          'token': token,
          'user': user.username,
          'userid': user.id
      }
  ```

- url表

  ```python
  from project_cs.views import Reg,RegisterView
  #用来登录时候的验证,并返回token
  from rest_framework_jwt.views import obtain_jwt_token
  
  urlpatterns = [
      path('admin/', admin.site.urls),
      #使用 obtain_jwt_token方法时候，会自动使用rest_framework_jwt进行登录信息验证
      path('get_token/', obtain_jwt_token), #注意要使用post请求方式
  
      
      path('reg/', Reg.as_view()),
      path('regs/', RegisterView.as_view()),
  ]
  ```

- ser序列化器

  ```python
  from rest_framework_jwt.settings import api_settings
  from rest_framework import serializers
  from project_cs.models import User
  
  class UserSerializer(serializers.Serializer):
      username = serializers.CharField()
      password = serializers.CharField()
      phone = serializers.CharField()
      token = serializers.CharField(read_only=True)
  
      def create(self, data):
          user = User.objects.create(**data)
          user.set_password(data.get('password'))
          user.save()
          #token生成国产车
          # 补充生成记录登录状态的token
          jwt_payload_handler = api_settings.JWT_PAYLOAD_HANDLER
          jwt_encode_handler = api_settings.JWT_ENCODE_HANDLER
          #往添加token的数据
          payload = jwt_payload_handler(user) #这里需要修改为自己的数据
          #生成对token进行加密
          token = jwt_encode_handler(payload)
  
          #往user数据里添加token，返回时好带上token
          user.token = token
          return user
  ```

  ---

### JWT 登录验证的扩充

- 自定义验证方式：要求手机或者邮箱也可作为登陆手段

  ```python
  #settings.py
  #配置自定义验证方式
  AUTHENTICATION_BACKENDS = [
      'userapp.views.UsernameMobileAuthBackend',
  ]
  ```

- 视图

  ```python
  from django.db.models import Q
  from django.contrib.auth.backends import ModelBackend #验证基类
  
  class UsernameMobileAuthBackend(ModelBackend):
      #重写authenticate自定义验证方式  
      def authenticate(self, request, username=None, password=None, **kwargs):
          #用户名或手机号都可以登录
          user = MyUser.objects.get(Q(username=username) | Q(phone=username))
          #如果用户密码都正确返回响应
          if user is not None and user.check_password(password):
              return user
  ```

  ---

### JWT 权限验证

  - 局部权限

    ```python
    from rest_framework.permissions import IsAuthenticated,AllowAny,IsAdminUser
    #可以用在用户登录以后，根据当用户的权限判断是否可以访问当API接口
    permission_classes = [IsAuthenticated]
    
    
    #django rest_framework jwt 自带三种权限  IsAuthenticated ,AllowAny,IsAdminUser
    #只有登录才可以访问该接口
    IsAuthenticated 
    #(也就是说前端需要给后端传递一个(Authorization:JWT token)则为已经登陆，反之没有登录)
    
    #所有人都能访问
    AllowAny
    #(带不带token都可以访问)
    
    #只有管理员才能访问
    IsAdminUser
    #(is_staff为1可以访问)
    #只有登录以后才可以访问该接口
    
    
    
    # 简单例子   
    class UserList(APIView):
        #局部权限 可以再自定义的API里加入
        permission_classes = [IsAuthenticated]  # 接口中加权限
        def get(self,request, *args, **kwargs):
            print(request.META.get('HTTP_AUTHORIZATION', None))
            return Response({'name':'zhangsan'})
        def post(self,request, *args, **kwargs):
            return Response({'name':'zhangsan'})
    ```

  - settings配置以及全局权限

    ```python
    #修改验证方式  改为JSONWebTokenAuthentication  （jwt的验证方式）
    REST_FRAMEWORK = {
        # 身份认证
        'DEFAULT_AUTHENTICATION_CLASSES': (
            'rest_framework_jwt.authentication.JSONWebTokenAuthentication',
            'rest_framework.authentication.SessionAuthentication',
            'rest_framework.authentication.BasicAuthentication',
        ),
        #全局配置JWT验证设置
    # 'DEFAULT_PERMISSION_CLASSES': (
    #             'rest_framework.permissions.IsAuthenticated',
    #         ),
    }
    ```

    ---

### JWT权限扩充

    ```python
    from rest_framework.permissions import BasePermission
    class SVIPPermission(BasePermission):
        message = '必须高级会员才能访问'
    
        def has_permission(self,request,view):
            #这里的user.id  可以换城你定义的字段如 user.type
            if request.user.id != 1:  
                return False
            return True
        
      #如何调用自定义jwt扩充的权限
    permission_classes = [SVIPPermission]  #添加自己的定义权限的名
    ```
