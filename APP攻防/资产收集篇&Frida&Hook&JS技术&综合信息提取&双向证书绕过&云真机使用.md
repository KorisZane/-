## 云真机使用

https://www.gc.com.cn/

## Firda HOOK技术

一款易用的跨平台Hook工具，Java层到Native层的Hook无所不能，是一种动态的插桩工具，可以插入代码到原生App的内存空间中，动态的去监视和修改行为，原生平台包括Win、Mac、Linux、Android、iOS全平台

### 安装

```
pip install frida
pip install frida-tools
```

https://github.com/frida/frida/releases

下载与本地版本一致的frida.server，选择相应系统，server版

```
adb root
adb push frida-server-17.6.2-android-x86_64 /data/local/tmp
adb shell
cd /data/local/tmp
chmod 777 frida-server
chmod 777 /data/local/tmp
./frida-server-17.6.2-android-x86_64
```

```
frida-ps -U 查看开启进程确保服务连接
frida -U -l 注入脚本 注入app进程号
```

## 综合资产提取

https://github.com/dwisiswant0/apkleaks
https://github.com/MobSF/Mobile-Security-Framework-MobSF

# frida hook

## 一、最最基础的 Hook 脚本骨架（先背下来）

```
Java.perform(function () {
    console.log("Frida Hook Start");
});
```

⚠️ 记住：

- **所有 Java Hook 必须写在 `Java.perform` 里**
    
- 不然 99% 报错
    

---

## 二、Hook 一个 Java 方法（核心入门）

### 例子：Hook `java.lang.String.toString()`

```
Java.perform(function () {

    var StringCls = Java.use("java.lang.String");

    StringCls.toString.implementation = function () {
        var ret = this.toString();
        console.log("toString called ->", ret);
        return ret;
    };

});

```
### 这里发生了什么（非常重要）

| 代码                | 意思                 |
| ----------------- | ------------------ |
| `Java.use`        | 拿到 Java 类          |
| `.implementation` | 重写这个方法             |
| `this`            | 当前对象               |
| `return ret`      | 不 return = 原函数直接废掉 |

---

## 三、Hook 带参数的方法（最常见）

### Java 方法：

```
boolean check(String s, int i)
```

### Frida 写法：

```
Java.perform(function () {

    var Cls = Java.use("com.xxx.xxx.Checker");

    Cls.check.implementation = function (s, i) {

        console.log("s =", s);
        console.log("i =", i);

        var ret = this.check(s, i);
        console.log("ret =", ret);

        return ret;
    };

});

```

⚠️ **参数顺序、个数必须一模一样**

---

## 四、Hook 重载方法（新手最容易死这）

### Java 里有重载：

```
check(String s)
check(String s, int i)
```
### Frida 必须指定 `.overload`

```
Cls.check
    .overload("java.lang.String", "int")
    .implementation = function (s, i) {

        console.log(s, i);
        return true;   // 直接改逻辑

    };
```

📌 **不知道参数？用 jadx 看，或者用 frida-trace**

---

## 五、直接改返回值（绕校验神器）

```
Cls.isVip.implementation = function () {
    console.log("isVip called");
    return true;
};
```

常见用途：

- VIP 判断
    
- 登录状态
    
- Root 检测
    
- Debug 检测
    

---

## 六、Hook 构造函数（`$init`）

### Java：

```
new User(String name)
```
### Frida：

```
var User = Java.use("com.xxx.User");

User.$init
    .overload("java.lang.String")
    .implementation = function (name) {

        console.log("User name =", name);

        return this.$init(name);
    };
```

## Hook示例

只校验host

```
 final HostnameVerifier hostnameVerifier = new HostnameVerifier() { // from class: ddns.android.vuls.activities.net.HttpsURLConnectionActivity.6
  
            @Override // javax.net.ssl.HostnameVerifier
  
            public boolean verify(String hostname, SSLSession session) {
  
                Log.e("NetAcitivty", hostname);
  
                HostnameVerifier hv = HttpsURLConnection.getDefaultHostnameVerifier();
  
                Boolean result = Boolean.valueOf(hv.verify("www.google.com", session));
  
                return result.booleanValue();
  
            }
  
        };
```

hook脚本

```
// universal_bypass.js
Java.perform(function() {
    console.log("[*] Starting universal hooks for doCheckHost...");
    
    // 方法 1: Hook HttpsURLConnection.getDefaultHostnameVerifier()
    var HttpsURLConnection = Java.use('javax.net.ssl.HttpsURLConnection');
    HttpsURLConnection.getDefaultHostnameVerifier.implementation = function() {
        console.log("[*] HttpsURLConnection.getDefaultHostnameVerifier() called");
        
        // 创建一个自定义的 HostnameVerifier，始终返回 true
        var HostnameVerifier = Java.use('javax.net.ssl.HostnameVerifier');
        var CustomVerifier = Java.registerClass({
            name: 'com.example.CustomVerifier',
            implements: [HostnameVerifier],
            methods: {
                verify: function(hostname, session) {
                    console.log("[*] CustomVerifier.verify() called");
                    console.log("[*] Hostname: " + hostname);
                    console.log("[*] Always returning true");
                    return true;
                }
            }
        });
        
        var verifier = CustomVerifier.$new();
        console.log("[*] Returning custom verifier");
        return verifier;
    };
    console.log("[*] HttpsURLConnection.getDefaultHostnameVerifier() hooked successfully!");
});
```

```
frida-ps -U 查看进程号 
frida-ps -U -l 注入脚本 进程号
```


