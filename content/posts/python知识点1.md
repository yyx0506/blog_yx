
---
title: "Python知识点1"
date: 2022-12-01
categories:
- tranquilpeak
- features
tags:
- python
# keywords:
# - python

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->



### 1.网络模型
![](https://cdn.nlark.com/yuque/0/2022/png/8367699/1648630958169-8f3bf03a-13bb-4851-88e2-b5972ea9cbb9.png?x-oss-process=image%2Fresize%2Cw_937%2Climit_0#crop=0&crop=0&crop=1&crop=1&from=url&id=pbKI4&margin=%5Bobject%20Object%5D&originHeight=513&originWidth=937&originalType=binary&ratio=1&rotation=0&showTitle=false&status=done&style=none&title=)
**http(s) 应⽤层 **
**socks 会话层 **
**tcp、udp 传输层 **
**ip ⽹络层**
### 1.简述HTTP协议和HTTPS协议的区别
HTTP协议传输的数据都是未加密的，也就是明文的，因此使用HTTP协议传输隐私数据信息非常不安全，为了保证这些隐私数据的加密传输，于是网景公司设计了SSL协议用于对HTTP协议传输的数据进行加密，从而诞生了HTTPS，简单来说HTTPS协议就是SSL+HTTP协议构建的，他们的区别有：
A：TPHTTPS协议需要到ca申请证书，一般免费证书较少，因此需要一定的费用
B：HTTOP信息是明文传递的，HTTPS则是具有安全性的SSL加密协议传输的
C：使用完全不同的连接方式，端口也不一样，HTTP通常是80端口，THTTPS是443
D：HTTP的连接是无状态的，HTTPS是由SSL+HTTP协议构建的可进行加密传输的协
### 2.三次握手四次挥手
客户端向服务器发送消息报文请求连接
服务器收到请求后，回复报文确定可以连接
客户端收到回复，发送最终报文连接建立

主动方发送报文请求断开连接
被动方收到请求后，立即回复，表示准备断开
被动方准备就绪，再次发送报文表示可以断开
主动方收到确定，发送最终报文完成断开
### 3.在网页里输入域名，按下确定的过程后，发生了什么
域名解析
域名解析检查顺序为：浏览器自身DNS缓存---》OS自身的DNS缓存--》读取host文件--》本地域名服务器--》权限域名服务器--》根域名服务器。如果有且没有过期，则结束本次域名解析。域名解析成功之后，进行后续操作
2.tcp3次握手建立连接
3.建立连接后，发起http请求
4.服务器端响应http请求，浏览器得到到http请求的内容；
5.浏览器解析html代码，并请求html代码中的资源
6. 浏览器对页面进行渲染，展现在用户面前

### 4.TCP协议和IP协议有啥区别？
我们以现实生活中货物运输举例，现实生活中货物运输都是把货物打包好封成一件一件的去运输，网络世界中也是如此，IP协议就是规定了数据传输时候的单元和格式，比作货物运输就是规定了货物打包时候的尺寸和包装的程序，除了这以外IP还定义了数据包的递交办法和路由选择，但是IP所传输的数据是单向的，也就是说IP只管发，对方收没收到不知道，TCP就不一样，它提供了可靠的面向是对象的数据流传输服务的股则和约定，简单的说就是TCP模式中，对方发一个数据包给你，你要发一个确认数据包给对方
### 5.scrapy的工作流程和去重机制
首先spiders将需要发送请求的url通过scrapyEngine(引擎)交给Schedule(调度器)，然后Schedule(调度器)处理后，经ScrapyEngine，DownloaderMiddlewares交给Downloader，Downloader向互联网发送请求，并接受下载响应，将响应经ScrapyEngine，SpiderMiddlewares交给Spiders，然后Spiders处理response，提取数据并将数据经scrapyEngine交给itemPipeline保存(可以是本地，可以是数据库)，最后提取url重新经scrapyEngine交给scheduler经下一个循环，直到无url请求程序停止结束为止
去重机制：scrapy是通过hashlib算法转成长度一致的url，然后通过set集合去重的，去重的中间件在scrapy的dupefilters.py文件中：--->有个去重器RFDupeFilter()，有个函数叫request_seen()，每次执行之前会执行request_seen()函数生成一个指纹，然后就是去重器中的self.fingerprints=set()，通过上句代码就实现了set集合去重了
### 6. 线程，进程，协程你是怎么理解的，一般在什么场景中使用他们
进程：一个运行的程序或代码就是一个进程，进程是系统进行资源分配的最小单位，拥有自己的内存空间，他们之间互相独立，数据不共享，使用场景：cpu密集型中操作，比如对视频高清解码，需要到大量的计算
线程：调度执行的最小单位，不能独立存在，依赖进程的存在而存在，一个进程至少一个线程，多个线程之间内存共享，使用场景：IO密集型操作，读写数据较多时
携程：用户态的轻量级线程，调度用户控制，拥有自己的寄存器上下文的栈，使用场景：python中的多线程
### 7.使用线程池开启多线程应该注意什么问题
线程池开启多线程的时候，会同步发送类似请求给服务器，导致服务器瞬间压力，正确的做法是逐步开启线程，最终达到最大线程数，而不是同步开启，同步开启会导致风控机制的快速触发，再就是应该注意线程锁，变量控制以及线程阻塞问题
### 8.造成线程阻塞的原因可能有哪些
A：线程执行了Thread.sleep()方法，当前线程放弃CPU，睡眠一段时间后在执行
B：线程执行了一段同步代码，但是尚且无法获得相关的同步锁，只能进入阻塞状态，等到获取到了同步锁，才能恢复执行
C：线程执行了一个对象的wait()方法，直接进入阻塞状态
D：线程在执行某些IO操作，因为等待相关的资源而进入阻塞状态，比如监听键盘，但是目前没有输出值所以进入阻塞状态
### 9.分布式
只问原理和实现，scrapy比较熟悉可以跟他吹scrapy-redis，延伸去说feapder，这个面试到现在的公司没有一个知道，侧面展示你的技术紧跟潮流
都不吹的话就说自己没用框架实现，用一个简单生产者消费者的模式，两个进程，基于redis一个存，存的进程可以里面写异步加快连接获取，另外一个进程一个取，下面的spiders做个多线程也能简单地满足大部分公司的需求，不是框架的缘故自己好扩展
### 10. 异步协程，线程
主要问题aiohttp和asyncio
线程的锁是个咋回事，python因为锁的缘故一直被诟病假的多线程，看面试官深不深入跟你吹这个锁，不过基本点到为止，锁的好处跟坏处说完也就差不多了
### 11.请说明http协议中，get请求和post请求的区别
GET方法
GET是获取的意思，顾名思义就是获取信息。
GET是默认的HTTP请求方法。
GET方法把参数通过key/value形式存放在URL里面，如果参数是英文数字原样显示，如果是中文或者其他字符加密（Base64）URL长度一般有限制所以GET方法的参数长度不能太长。由于参数显示再地址栏所以不安全，一般需要保密的请求不使用GET。
POST方法
POST是邮件的意思，顾名思义就像一封信一样将参数放在信封里面传输。它用于修改服务器上的数据，一般这些数据是应该保密的，就像信件一样，信的内容只能收信的人看见。例入当用户输入账号和密码登录时账号和密码作为参数通过HTTP请求传输到服务器，这时候肯定不能用GET方法将账号密码直接显示再URL上，这时候就应该用POST方法保证数据的保密性。

1.  get是从服务器上获取数据，post是向服务器传送数据。 
2.  GET请求把参数包含在URL中，将请求信息放在URL后面，POST请求通过request body传递参数，将请求信息放置在报文体中。 
3.  get传送的数据量较小，不能大于2KB。post传送的数据量较大，一般被默认为不受限制。但理论上，IIS4中最大量为80KB，IIS5中为100KB。 
4.  get安全性非常低，get设计成传输数据，一般都在地址栏里面可以看到，post安全性较高，post传递数据比较隐私，所以在地址栏看不到， 如果没有加密，他们安全级别都是一样的，随便一个监听器都可以把所有的数据监听到。 
5.  GET请求能够被缓存，GET请求会保存在浏览器的浏览记录中，以GET请求的URL能够保存为浏览器书签，post请求不具有这些功能。 
6.  HTTP的底层是TCP/IP，GET和POST的底层也是TCP/IP，也就是说，GET/POST都是TCP链接。GET和POST能做的事情是一样一样的。你要给GET加上request body，给POST带上url参数，技术上是完全行的通的。 

7.GET产生一个TCP数据包，对于GET方式的请求，浏览器会把http header和data一并发送出去，服务器响应200（返回数据）；POST产生两个TCP数据包，对于POST，浏览器先发送header，服务器响应100 continue，浏览器再发送data，服务器响应200 ok（返回数据），并不是所有浏览器都会在POST中发送两次包，Firefox就只发送一次。
### 12.有用异步自实现爬虫框架吗？有的话说出其架构思路，可控方法，有缺点
### 13.请简述javascript中原型链以及常见的javascript指纹检测对象有哪些？
### 14.常见的反扒产品？反扒参数？反扒措施有哪些
### 15.android逆向中，androidMainFest.xml文件的主要作用，如何找到启动函数入口
AndroidManifest.xml是Android应用的入口文件，它描述了package中暴露的组件（activities, services, 等等），他们各自的实现类，各种能被处理的数据和启动位置。 除了能声明程序中的Activities, ContentProviders, Services, 和Intent Receivers,还能指定permissions和instrumentation（安全控制和测试）。
### 16.justtrustme的原理
JustTrustMe 一个用来禁用、绕过 SSL 证书检查的基于 Xposed 模块。
JustTrustMe 是将 APK 中所有用于校验 SSL 证书的 API 都进行了 Hook，从而绕过证书检
查的
### 17.了解过ssl-pinning吗？
客户端会做两个证书间的一次性校验，那么就通过hook的方式将此次校验的结果返回true或者干脆不让其做校验
### 18.ssl-pinning 的客户端证书是怎末获取的？
checkServerTrusted方法中取出证书链中对应网站的证书（也就是第一个），和预置的证书去判断是否相同，如果不同则抛出异常。
这里可以不校验证书，只校验证书的部分信息（其实，安装了openssl环境的话，基本上所有服务器的ssl证书都能拿到，所以安全性问题不存在的~~）
### 19.常见的加固方案，如何脱壳
apk加壳原理：壳apk的dex文件读取源apk的 classes.dex 文件，加密后写入新的dex文件，组成新的apk
加壳apk加载过程：新apk被点击, 壳apk会把源classes.dex文件读取出来，解密后, 把源classes.dex给加载起来
frida脱壳思路： 在壳apk把源classes.dex文件读取出来解密之后, 将源classes.dex文件内容从内存中dump出来, 写入自定义dex文件中
### 20.注入框架和hook框架的原理
### 21.app逆向
说个简单的流程，尽量把每一步会遇到的问题和解决办法稍微带一下
so进洞绑定如何判断（这个我回答的就一个看so文件是否有java名，jni里找下绑定，要是混淆了，hook RegisterNatives，直接unidbg里调也可以）
大部分公司不需要你直接so底层算法直接搞出来，你会调用就能满足需求了，目前我没遇到非要我搞定底层，认怂的时候就认怂，底层这块认怂没关系
frida、xposed哪个用了说那个，hook的原理,frida的原理，hook到的是个传参是object该如何处理，如何找到加密入口
破解后你是如何处理，java层的和native层的各自说下破解后是咋调用的，java层改写成python或者变成jar包，so的话rpc或者unidbg，unicorn
### 22.ida常用命令
shift + f12 查看so文件中的所有常量字符串的值
f5:汇编转 c
/ :添加注释
N:变量重命名
X:查看某变量的所有引用
H:常量改16进制 ，便于查看是否加密算法的初始化链接变量的一致
### 23：apktool
反编译apk包： 
 apktool d test.apk
重打包成apk： 
 apktool b test
### 24：smali: 
 apktool d 反编译代码成smali文件：smali一种类似汇编格式 
 可以修改/增删smali代码，重打包成apk后执行 
 结合java代码看smali代码
25：APP简单启动顺序：点击APP启动--->执行application(oncreate方法)------>执行main 
activity(oncreate方法)---->启动成功
25
### 25.openMemory的脱壳原理 
# 思想，此方法是根据openMemory的地址来计算出此hook app运行时dex文件的地址，dex文件的地址加上32个字节，就是dex文件的大小
# 根据dex文件的大小和地址 就可以得到dex文件
### 26.jadx反编译高度混淆找不到参数 
使用objection内存漫游
### 27：GIL 
在Python 程序的工作示例。其中，Thread 1、2、3 轮流执行，每一个线程在开始执行时，都会锁住 GIL，以阻止别的线程执行；同样的，每一个线程执行完一段后，会释放 GIL，以允许别的线程开始
GIL 的功能是：在 CPython 解释器中执行的每一个 Python 线程，都会先锁住自己，以阻止别的线程执行。

