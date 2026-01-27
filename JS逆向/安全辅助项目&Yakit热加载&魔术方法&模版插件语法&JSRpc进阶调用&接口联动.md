## Yakit热加载

https://yaklang.com/products/Web%20Fuzzer/fuzz-hotpatch

以数据包发包前和发包后的数据修改功能

1. 热加载的模板内容
	```
	格式：{{yak(函数名|参数名)}}
	
	如密码：{{yak(upper|{{x(pass_top25)}})}}
	```
调用执行函数

```
upper = func(s) {
// 传入的参数，类型为字符串，返回值可以是字符串或数组
return s.Upper()
}
```

![](yr1.png)

```
{{yak(upper|{{x(pass_top25)}})}}
```

函数声明 x(pass_top25) 是传入的参数(yakit自带x(pass_top25))

下面的是函数的定义，传入x(pass_top25)，返回x(pass_top25)

## 案例1(靶场案例)

算法分析 [[Sign签名&绕过技术&算法可逆&替换库模拟发包&堆栈定位&特征搜索&安全影响]] 案例一

函数定义

```
func sign(user, pass){
    return codec.EncodeToHex(codec.HmacSha256("1234123412341234", f`username=${user}&password=${pass}`)~)
}

getSign = result => {
    pairs := result.SplitN("|", 2)
    dump(pairs)
    return sign(pairs[0], pairs[1])
}
```

调用执行

```
{{yak(getSign|admin|{{x(pass_top25)}})}}
```

在以图片的方式发包

![](yr2.png)

选择fuzz设置变量，在数据包中替换变量

## 案例二

AES CBC 加密，iv，key已数据包传输为准

函数定义

```
aescbc = result => {
    result = codec.AESCBCEncryptWithPKCS7Padding(
        codec.DecodeHex('31323334313233343132333431323334')~,
        result,
        codec.DecodeHex('bfeb9fdf041beaa31a1f136700f5e25c')~
        )~
    return codec.EncodeBase64(string(result))
}
```

调用执行

```
{{yak(aescbc|{"username":"admin","password":"{{x(pass_top25)}}"})}}
```

## 魔术方法

```
// beforeRequest 允许发送数据包前再做一次处理，定义为 func(origin []byte) []byte

beforeRequest = func(req) {
	return []byte(req)
}

// afterRequest 允许对每一个请求的响应做处理，定义为 func(origin []byte) []byte

afterRequest = func(rsp) {
	return []byte(rsp)
}
```

## JSRpc热加载

```
{{yak(jsrpcReq|{{payload(pass_top25)}})}}
```

```
jsrpcReq = func(origin /*string*/) {

// JSrpc的group

group = "zzz";

// jsrpc的action

action = "decrypt";

if (origin[0] == "{") {

rsp, rep = poc.Post(

"http://127.0.0.1:12080/go",

poc.replaceBody("group=" + group + "&action=" + action + "&param=" + json.dumps(origin), false),

poc.appendHeader("content-type", "application/x-www-form-urlencoded")

)~

return json.loads(rsp.GetBody())["data"];

} else {

rsp, rep = poc.Post(

"http://127.0.0.1:12080/go",

poc.replaceBody("group=" + group + "&action=" + action + "&param=" + codec.EncodeUrl(origin), false),

poc.appendHeader("content-type", "application/x-www-form-urlencoded")

)~

return json.loads(rsp.GetBody())["data"];

}

}

// beforeRequest 允许在每次发送数据包前对请求做最后的处理，定义为 func(https bool, originReq []byte, req []byte) []byte

// https 请求是否为https请求

// originReq 原始请求

// req 请求

beforeRequest = func(https, originReq, req) {

// 我们可以将请求进行一定的修改

postParams = poc.GetAllHTTPPacketPostParams(req /*type: []byte*/)

encryptedParam = jsrpcReq(postParams["encryptedData"])

req = poc.ReplaceHTTPPacketPostParam(req, "encryptedData", encryptedParam)

// 将修改后的请求返回

return []byte(req)

}

• poc.GetAllHTTPPacketPostParams 从传入的req数据包中获取所有Post参数

• jsrpcReq 将 encryptedData 的值发送到jsRpc的API中，返回值是加密后的参数值

• poc.ReplaceHTTPPacketPostParam 替换req中Post参数名为encryptedData的参数值，然后将修改后的数据包返回
```