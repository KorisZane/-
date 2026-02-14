## 脱壳

未加壳的app：
Java+安卓 -> APK -> dex文件 -> 运行

加壳的app：
Java+安卓 -> APK -> 360/腾讯等（SO算法） -> 内存 -> 运行

### 脱壳的本质

在内存中获取，让APP运行在手机中，内存中加载文件内容写回到dex文件

## 查壳

https://github.com/sulab999/AppMessenger

## frida-dexdump

root 文件读写 USB调试

连接frida

```
adb forward tcp:27042 tcp:27042
连接判断：frida-ps -U frida-ps -R
frida-dexdump -U -f "包名" （-f 参数会尝试重新启动应用，容易超时）
如果启动黑屏手工启动APP后执行：frida-dexdump -U -F "com.heiyan.reader"
如果启动黑屏手工启动APP后执行：frida-dexdump -U -F "com.flutter324.cbxxk.xr0e8m"
如果启动黑屏手工启动APP后执行：frida-dexdump -U -F "com.auto_play"
如果启动黑屏手工启动APP后执行：frida-dexdump -U -F
```

在用户目录下出现脱壳文件
传到模拟器用MT修复，压缩
用反编译工具反编译
## 面具+LSP+Fundex2(真机)

[https://github.com/Xposed-Modules-Repo/com.zhenxi.fundex2](https://github.com/Xposed-Modules-Repo/com.zhenxi.fundex2)

- 真机测试报告
关于Android10及以上 frida-实验报告v2-成功版.PDF

- 存储安全测试
APP内敏感数据存储导致的泄露
内部文件，外部文件，数据库文件，日志存储等