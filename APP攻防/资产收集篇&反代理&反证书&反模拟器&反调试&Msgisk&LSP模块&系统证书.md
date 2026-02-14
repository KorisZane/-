![](dl.jpg)
![](dl2.jpg)
![](dl3.jpg)
![](dl4.jpg)

## 没有限制过滤的抓包问题

1、抓不到-工具证书没配置好
2、抓不到-app走的不是https

证书安装参考 https://blog.csdn.net/u012556659/article/details/147297337

### 安装系统证书

1、开启root权限
2、开启磁盘可写入

```
openssl x509 -inform DER -in cacert.der -out cacert.pem                //转换格式
openssl x509 -inform PEM -subject_hash_old -in cacert.pem          //计算哈希
mv cacert.pem 9a5ba575.0

adb.exe root
adb.exe devices
adb.exe push 9a5ba575.0 /sdcard/               
adb.exe shell
    mount -o rw,remount /
mount -o rw,remount /system
chmod 777 /system
mount -o remount -o rw /
cp /sdcard/9a5ba575.0 /system/etc/security/cacerts/
chmod 644 /system/etc/security/cacerts/9a5ba575.0
reboot
```

## 反代理

1. 用APP工具设置-Postern
2. 用PC工具设置-Proxifier
3. 用自带的App抓包
4. 逆向删反代码重打包

https://github.com/wanghongenpin/proxypin/releases

## 反证书

安装面具和Lposed框架

1、单向-XP或LSP等
2、双向-看下课补充
3、逆向删反代码重打包

LSP模块安装：（Magisk+LSPosed等）

https://github.com/LSPosed/LSPosed

https://github.com/topjohnwu/Magisk

[https://blog.csdn.net/danran550/article/details/132256027](https://blog.csdn.net/danran550/article/details/132256027)

## 反模拟器

1、用真机
2、模拟器模拟真机
3、逆向删反代码重打包