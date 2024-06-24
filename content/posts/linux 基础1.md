---
title: "linux 基础1"
date: 2021-12-28
categories:
- tranquilpeak
- features
tags:
- linux
# keywords:
# - linux
# - grep
# - crontab
# - netstat
# - df

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->

### linux 基础1

#### 1:服务器与本地之间文件互传 

```
将某个文件传到 对应的服务器内的跟目录下 scp id_rsa.pub root@xxx.xx.xxx.xx:~/
将某个文件从 服务器传到本地 scp -r root@×××.×××.×××.×××:/data/ /home/myfile/  其中 -r 代表这个是文件夹
```

#### 2: grep

```
一般配合cat查询日志 比如 cat logs/xxxx.log |grep  xxx. 查询 xxxx.log 日志的含有xxx的内容
如果有一些命令忘记了也可以去查询一下 例如 history |grep xxx
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +3|head   cpu占用最多的10个进程
ps aux|head -1;ps aux|grep -v PID|sort -rn -k +4|head  内存占用最多的10个进程
```

#### 3:crontab

```
 crontab -l 查询当前 crontab 的书写 
 crontab -e 修改当前 crontab 文件
 *           *        *        *        *        代表着分，时，天，月，星期
 30 * * * * cd /tmp/ && rm -rf *log*   代表每30分钟启动一次
 0 10 * * * supervisorctl restart xxx:  代表 每天10天启动一次
 0 */2 * * * supervisorctl restart xxxxxxx: 代表 每两个小时启动一次
```

#### 4: netstat

```
netstat -tlunp |grep 9362 查看某个端口有没有服务启动
netstat -s # 显示所有端口的统计信息
netstat -st # 显示所有TCP的统计信息
netstat -su # 显示所有UDP的统计信息
netstat -p 显示 PID 和进程名称
```

#### 5:df

```
du -sh * 查看 当前文件夹下问价的各个大小 
df -h 查看 当前服务器下各个盘的使用情况
```

