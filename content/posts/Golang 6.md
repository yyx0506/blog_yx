
---
title: "Golang  time"
date: 2022-08-30
categories:
- tranquilpeak
- features
tags:
- golang
# keywords:
# - golang
thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### Golang 6 time

#### 1 time

时间和日期是我们编程中经常会用到的，在golang中time包提供了时间的显示和测量用的函数。

```
timeObj := time.Now()
fmt.Println(timeObj)
year := timeObj.Year()
month := timeObj.Month()
day := timeObj.Day()
fmt.Printf("%d-%02d-%02d \n", year, month, day)

结果 ----- 
2022-09-11 20:44:07.221505 +0800 CST m=+0.000129922
2022-09-11 
格式化日期 

时间类型有一个自带的方法 Format进行格式化
需要注意的是Go语言中格式化时间模板不是长久的 Y-m-d H:M:S
而是使用Go的诞生时间 2006年1月2日 15点04分

获取当前时间戳
timeObj3 := time.Now()
// 获取毫秒时间戳
unixTime := timeObj3.Unix()
// 获取纳秒时间戳
unixNaTime := timeObj3.UnixNano()
fmt.Println(unixTime, unixNaTime)

时间戳转日期字符串
// 时间戳转换年月日时分秒（一个参数是秒，另一个参数是毫秒）
var timeObj4 = time.Unix(1595289901, 0)
var timeStr = timeObj4.Format("2006-01-02 15:04:05")
fmt.Println(timeStr)

日期字符串转换成时间戳
var timeStr2 = "2020-07-21 08:10:05";
var tmp = "2006-01-02 15:04:05"
timeObj5, _ := time.ParseInLocation(tmp, timeStr2, time.Local)
fmt.Println(timeObj5.Unix())

时间间隔
time.Duration是time包定义的一个类型，它代表两个时间点之间经过的时间，以纳秒为单位。time.Duration表示一段时间间隔，可表示的最大长度段大约290年。
```

#### 2: 时间操作函数

我们在日常的编码过程中可能会遇到要求时间+时间间隔的需求，Go语言的时间对象有提供Add方法如下

```
func (t Time) Add(d Duration)Time
```

例如

```
// 时间相加
now := time.Now()
// 当前时间加1个小时后
later := now.Add(time.Hour)
fmt.Println(later)
```

同理的方法还有：时间差、判断相等

#### 3: 定时器

方式1：使用time.NewTicker（时间间隔）来设置定时器

```
// 定时器, 定义一个1秒间隔的定时器
ticker := time.NewTicker(time.Second)
n := 0
for i := range ticker.C {
    fmt.Println(i)
    n++
    if n>5 {
        // 终止定时器
        ticker.Stop()
        return
    }
}
```

方式2：time.Sleep(time.Second)来实现定时器

```
for  {
    time.Sleep(time.Second)
    fmt.Println("一秒后")
}
```

tips : time 模块功能很多，后续会出一个专题介绍