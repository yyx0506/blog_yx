---
title: "nats 高级特性 "
date: 2022-11-18
categories:
- tranquilpeak
- features
tags:
- nats
- python
- golang
# keywords:
# - nats

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


### nats 高级特性 

#### 1: nats 在python 端的高级特性

```
除了用传统意义上的mq功能外，就会借助nats来实现消息总线。这里的消息总线模式更像是微服务的另一种模型。
而基于消息总线的微服务设计是怎样？服务提供者监听nats mq topic，服务调用者直接把请求扔到mq topic，然后接收返回的结果？ 为什么是疑问？ nats大体是可以分为三个模式，跟大多mq一样的pubsub模式，还有queue模式，跟其他mq不一样的request reply模式。  mq一般可以理解为异步消费，也就是说producer只需要把任务丢到mq就完事了，但是如果你要实现同步，那么producer和provider两端需要定义好 任务信息topic和结果topic。然后producer pub任务后，需要再sub订阅结果topic

这就是nats 独有的为微服务专门搞了一个request & reply。

下面用python 来演示一下
首先是生产者开启监听模式

class NatsProducer(object):

    sc = None
    topic = None

    async def msg_handler(self,msg):
        try:
            if msg.data:
                result = msgpack.unpackb(msg.data, raw=False)
                print(result)
                msginfo = {
                    '_type':1,
                    'data':'ok,ok',
                }
                print(msg.reply)
                await self.sc.publish(msg.reply, msgpack.packb(msginfo))

        except Exception as e:
            print(traceback.format_exc())


    async def run(self):
        self.topic = 'test.reply'
        self.sc = await NatsService.get()
        await self.sc.subscribe(self.topic, cb=self.msg_handler)
        while True:
            await asyncio.sleep(0.1)

然后接受者准备发送数据 并 自定义等待生产者的时间 默认是0.5秒 我们就设个60秒了


class NatsReplyConsumer(object):


    async def test_reply_consumer(self):
        while 1:
            try:
                t = time.time()
                nc = await NatsService.get()
                response = await nc.request('test.reply',
                                            msgpack.packb({'keyword': 'lol'}), timeout=60)
                print('request app_status', time.time() - t)
                print(msgpack.unpackb(response.data, raw=False))
            except Exception as e:
                print(e)
            await asyncio.sleep(4)


核心的关键就在于 这个reply 值 他用来 判断双方的接收值的 
```

#### 2 GO 使用nats

```
大家都知道 nats 使用go 语言写的 那么 go 是怎么使用 nats 的呢 我们来试一下

先去下载一下 nats的包 
go get -u -v github.com/nats-io/go-nats

下面是request 端的代码
package main

import (
	"encoding/json"
	"github.com/nats-io/go-nats"
	"log"
	"runtime"
	"time"
)

func main() {
	var url = "nats://121.5.58.115:4222"
	nc, err := nats.Connect(url, nats.UserInfo("nats", "P@ssw0rd"))
	if err != nil {
		log.Fatal("connect error")
	}
	nc.Subscribe("dalong", func(mess *nats.Msg) {
		log.Println(string(mess.Data), "from nats")
		result, _ := json.Marshal(mess)
		log.Println("the reply info is ", string(result))
	})
	message, err := nc.Request("dalong", []byte("dalong"), 1*time.Second)
	if err != nil {
		log.Println("get error, timeout", err)
	}
	log.Println("get data", string(message.Data))
	runtime.Goexit()
}


然后是reply 端的代码

package main

import (
	"log"

	"runtime"

	"encoding/json"

	"github.com/nats-io/go-nats"
)

func main() {
	var url = "nats://121.5.58.115:4222"
	nc, err := nats.Connect(url, nats.UserInfo("nats", "P@ssw0rd"))
	if err != nil {
		log.Fatal("connect error")
	}
	nc.Subscribe("dalong", func(mess *nats.Msg) {
		log.Println(string(mess.Data), "from nats")
		result, _ := json.Marshal(mess)
		log.Println("the reply info is ", string(result))
		nc.Publish(mess.Reply, []byte("dalong can help you"))
	})
	runtime.Goexit()
}

可以看到 我们也是用 reply 这个操作 那么这个reply 是怎么工作的呢 看下面一张图

```


![](/img/nats1.png)


request & reply的实现基本是在nats client端实现的。nats server对request & reply没做啥特别的东西。我们这里拿golang的nats client来分析下源码。go nats client的协程控制的很好，没有随意的滥开协程。subscribe自身没有并发控制，subscribe绑定了一个事件和回调方法后，会new一个waitForMsgs协程来回调，也就是说单个subscribe是同步阻塞的。 这个需要注意下，在go里nsq和kafka的sarama是模式并发模式的。



这个 我们分为三步来讲 

**1：nats client连接的过程**

```
可以看到 go 的官方源码 一个nats client的连接会开启2个协程，一个是用来读取数据，一个用来flush数据。 

// Process a connected connection and initialize properly.
func (nc *Conn) processConnectInit() error {

	// Set our deadline for the whole connect process
	nc.conn.SetDeadline(time.Now().Add(nc.Opts.Timeout))
	defer nc.conn.SetDeadline(time.Time{})

	// Set our status to connecting.
	nc.status = CONNECTING

	// Process the INFO protocol received from the server
	err := nc.processExpectedInfo()
	if err != nil {
		return err
	}

	// Send the CONNECT protocol along with the initial PING protocol.
	// Wait for the PONG response (or any error that we get from the server).
	err = nc.sendConnect()
	if err != nil {
		return err
	}

	// Reset the number of PING sent out
	nc.pout = 0

	// Start or reset Timer
	if nc.Opts.PingInterval > 0 {
		if nc.ptmr == nil {
			nc.ptmr = time.AfterFunc(nc.Opts.PingInterval, nc.processPingTimer)
		} else {
			nc.ptmr.Reset(nc.Opts.PingInterval)
		}
	}

	// Start the readLoop and flusher go routines, we will wait on both on a reconnect event.
	nc.wg.Add(2)
	go nc.readLoop()
	go nc.flusher()

	return nil
}
```

**nats request发送的过程**

针对request模式，一个连接只有一个waitForMsgs协程，nats通过sync.Once限制唯一。这里waitForMsgs的作用只是把msg发给request等待的chan而已。当我们使用bus的request方法时，他内部会生成一个接收结果的chan，接着在绑定Subscribe里绑定事件， 然后阻塞等待接收结果的chan。当远端的订阅者把结果返回时, waitForMsg会把msg传给上面的chan。 request接收chan，结束阻塞状态，返回。

```

// Request will send a request payload and deliver the response message,
// or an error, including a timeout if no message was received properly.
func (nc *Conn) Request(subj string, data []byte, timeout time.Duration) (*Msg, error) {
	if nc == nil {
		return nil, ErrInvalidConnection
	}

	nc.mu.Lock()
	// If user wants the old style.
	if nc.Opts.UseOldRequestStyle {
		nc.mu.Unlock()
		return nc.oldRequest(subj, data, timeout)
	}

	// Do setup for the new style.
	if nc.respMap == nil {
		nc.initNewResp()
	}
	// Create literal Inbox and map to a chan msg.
	mch := make(chan *Msg, RequestChanLen)
	respInbox := nc.newRespInbox()
	token := respToken(respInbox)
	nc.respMap[token] = mch
	createSub := nc.respMux == nil
	ginbox := nc.respSub
	nc.mu.Unlock()

	if createSub {
		// Make sure scoped subscription is setup only once.
		var err error
		nc.respSetup.Do(func() { err = nc.createRespMux(ginbox) })
		if err != nil {
			return nil, err
		}
	}

	if err := nc.PublishRequest(subj, respInbox, data); err != nil {
		return nil, err
	}

	t := globalTimerPool.Get(timeout)
	defer globalTimerPool.Put(t)

	var ok bool
	var msg *Msg

	select {
	case msg, ok = <-mch:
		if !ok {
			return nil, ErrConnectionClosed
		}
	case <-t.C:
		nc.mu.Lock()
		delete(nc.respMap, token)
		nc.mu.Unlock()
		return nil, ErrTimeout
	}

	return msg, nil
}
```

**nats reply订阅接收的过程**

订阅消费者返回结果的topic，把获取的结果扔到request等待的channel里。订阅者去直接调用subscribe事件的时候，每个subscribe的调用都会new一个waitForMsg协程来回调我们注册的方法。

简单说，同一个连接下，多个sub就多个并发。 

```
func (nc *Conn) createRespMux(respSub string) error {
	s, err := nc.Subscribe(respSub, nc.respHandler)
	if err != nil {
		return err
	}
	nc.mu.Lock()
	nc.respMux = s
	nc.mu.Unlock()
	return nil
}
这里代码太多了 就不截取这么多了
```

#### 3：Tips

 其实nats request& reply的原理很简单，官方代码的质量真的很高，很多我看了好久也没太明白，大家可以多看看