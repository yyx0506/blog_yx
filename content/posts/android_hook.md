---
title: "android——hook 1"
date: 2022-12-03
categories:
- tranquilpeak
- features
tags:
- frida
- app逆向
# keywords:
# - frida
# - app逆向

thumbnailImagePosition: left
# thumbnailImage: //d1u9biwaxjngwg.cloudfront.net/elements-showcase/vintage-140.jpg
---

<!--more-->


## 1.frida 
[https://github.com/frida/frida/releases](https://github.com/frida/frida/releases)
adb shell su -c "/data/local/tmp/frida-server"  # 一键启动
adb connect 127.0.0.1:62001  # 夜神
adb forward tcp:27042 tcp:27042
adb forward tcp:27043 tcp:27043
chmod 777 frida-server  # 赋权限
## 2.hook脚本的使用
frida -UF -l hook_vpn.py  # frida注入脚本至当前应用
frida -U -f pkgname -l  keystore_dump.js --no-pause  # spawn模式  -U：usb，  -f：包名，   -l：当前脚本，  --no-pause：立即执行
··· ···
## 3.r0capture
[https://github.com/r0ysue/r0capture](https://github.com/r0ysue/r0capture)

- Spawn 模式：python3 r0capture.py -U -f com.qiyi.video -v
- Attach 模式，抓包内容保存成pcap文件供后续分析：python3 r0capture.py -U com.qiyi.video -v -p iqiyi.pcap  （建议使用Attach模式，从感兴趣的地方开始抓包，并且保存成pcap文件，供后续使用Wireshark进行分析）
- 收发包函数定位：Spawn和attach模式均默认开启；可以使用python r0capture.py -U -f cn.soulapp.android -v >> soul3.txt这样的命令将输出重定向至txt文件中稍后过滤内容
- 客户端证书导出功能：默认开启；必须以Spawm模式运行；运行脚本之前必须手动给App加上存储卡读写权限；并不是所有App都部署了服务器验证客户端的机制，只有配置了的才会在Apk中包含客户端证书导出后的证书位于/sdcard/Download/包名xxx.p12路径，导出多次，每一份均可用，密码默认为：r0ysue，推荐使用[keystore-explorer](http://keystore-explorer.org/)打开查看证书。
## 4.FRIDA-DEXDump
[https://github.com/hluwa/FRIDA-DEXDump](https://github.com/hluwa/FRIDA-DEXDump)
pip3 install frida-dexdump
frida-dexdump -U -f com.app.pkgname
## 5.frida_dump
[https://github.com/lasting-yang/frida_dump](https://github.com/lasting-yang/frida_dump)
python dump_so.py soname
**dump android dex:**
frida -U --no-pause -f packagename - l dump_dex.js
## 5.Objection
[https://blog.csdn.net/qq_43181451/article/details/116224556](https://blog.csdn.net/qq_43181451/article/details/116224556)
[https://www.bilibili.com/video/BV1JA411A7hF](https://www.bilibili.com/video/BV1JA411A7hF)
[https://www.cnblogs.com/Snark/p/12347364.html](https://www.cnblogs.com/Snark/p/12347364.html) 
objection -g packagename explore  # 启动
env  # 查看环境
memory list modules #  列举so文件
android hooking watch class_method com.xxx.xxx.xxx.decrypt --dump-args --dump-backtrace --dump-return  # hook类的传入参数 返回值
jobs kill xxxx  # 退出上个hook
android hooking watch class com.xxx.xxx --dump-args --dump-backtrace --dump-return  # hook类有多少方法
android sslpinning disable  # 过大部分证书校验
android hooking list activities  # hook所有的类
android hooking list activities  |grep xxxx  # 过滤hook所有的类
## 6.fridaUiTools
[https://www.bilibili.com/video/BV1EQ4y127Kw](https://www.bilibili.com/video/BV1EQ4y127Kw)
[https://github.com/dqzg12300/fridaUiTools](https://github.com/dqzg12300/fridaUiTools)
python kmainForm.py
## 7.unidbg
