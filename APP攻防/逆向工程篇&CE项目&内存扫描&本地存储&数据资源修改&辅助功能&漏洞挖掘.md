## Cheat Engine

Cheat Engine 是一款开源的内存扫描和修改工具

1、CE修改器使用说明
2、APP内存修改（淘小说&壁纸）
3、APP本地修改（社交聊天记录）
4、重生后迪宝在骗子酒馆无敌于世

## 挖掘实例

[https://mp.weixin.qq.com/s/Gkou_-aA5Mk5qDReTxoDDQ](https://mp.weixin.qq.com/s/Gkou_-aA5Mk5qDReTxoDDQ)
[https://mp.weixin.qq.com/s/7JZPOVDrEyNCnVzY44k9yQ](https://mp.weixin.qq.com/s/7JZPOVDrEyNCnVzY44k9yQ)
[https://mp.weixin.qq.com/s/6wtQlPxw7Z25bt5HgNZJcw](https://mp.weixin.qq.com/s/6wtQlPxw7Z25bt5HgNZJcw)
[https://mp.weixin.qq.com/s/D7cb_NpMcGHH0dJuiUDcUQ](https://mp.weixin.qq.com/s/D7cb_NpMcGHH0dJuiUDcUQ)

## JEB调试

root模式
adb可调试
磁盘可写入
进入开发者模式开启USB调试

发编译调试的安装包，修改AndroidMainfest.xml文件里的application标签,添加
```
android:debuggable="true"
```

安装包添加共享文件夹
在jeb打开apk文件

Ctrl+B下断点
![](v1.png)

![点击start开始调试](v2.png)

![Local是属性值](v3.png)

### 嘟嘟牛算法逆向

![发现加密参数](dd1.png)

![反编译搜索user/login](dd2.png)

![反编译代码，查看代码逻辑](dd3.png)

![找到加密函数](dd4.png)

![动态调试，找到密钥和偏移量](dd5.png)

## CE修改手机内存

下载 https://www.cheatengine.org/downloads.php

```
adb push ceserver_x86_64 /data/local/tmp
adb shell
su root
cd /data/local/tmp/
chmod 777 ceserver_x86_64
./ceserver_x86_64
adb forward tcp:52736 tcp:52736
```

在CE上面连接
![](cc1.png)