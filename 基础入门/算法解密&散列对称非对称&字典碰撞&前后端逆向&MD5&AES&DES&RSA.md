密文-有源码直接看源码分析算法（后端必须要有源码才能彻底知道）

密文-没有源码1、猜识别 2、看前端JS（加密逻辑是不是在前端）

## 算法加密

### 单向散列加密MD5

方便存储，损耗低：加密/加密对于性能的损耗微乎其微。

单向散列加密的缺点就是存在暴力破解的可能性，最好通过加盐值的方式提高安全性，此外可能存在散列冲突。我们都知道MD5加密也是可以破解的。

常见的单向散列加密算法有：

MD5 SHA MAC CRC

*解密条件*：密文即可，采用碰撞解密，几率看明文复杂程度

### 对称加密AES

对称加密优点是算法公开、计算量小、加密速度快、加密效率高。

缺点是发送方和接收方必须商定好密钥，然后使双方都能保存好密钥，密钥管理成为双方的负担。

常见的对称加密算法有：

DES AES RC4

*解密条件*：密文及密钥偏移量，采用逆向算法解密，条件成立即可解密成功

### 非对称加密

非对称加密的优点是与对称加密相比，安全性更好，加解密需要不同的密钥，公钥和私钥都可进行相互的加解密。

缺点是加密和解密花费时间长、速度慢，只适合对少量数据进行加密。

常见的非对称加密算法：

RSA RSA2 PKCS

*解密条件*：密文和公钥或私钥，采用逆向算法解密，条件成立即可解密成功

## 识别特征

### MD5密文特点：

1、由数字“0-9”和字母“a-f”所组成的字符串

2、固定的位数 16 和 32位

解密需求：密文即可，但复杂明文可能解不出

### BASE64编码特点：

0、大小写区分，通过数字和字母的组合

1、一般情况下密文尾部都会有两个等号，明文很少的时候则没有

2、明文越长密文越长，一般不会出现"/""+"在密文中

AES、DES密文特点：

同BASE64基本类似，但一般会出现"/"和"+"在密文中

解密需求：密文，模式，加密Key，偏移量，条件满足才可解出

### AES、DES密文特点：

同BASE64基本类似，但一般会出现"/"和"+"在密文中

解密需求：密文，模式，加密Key，偏移量，条件满足才可解出

### RSA密文特点：

特征同AES,DES相似，但是长度较长

解密需求：密文，公钥或私钥即可解出


其他密文特点见：

1.30余种加密编码类型的密文特征分析（建议收藏）

https://mp.weixin.qq.com/s?__biz=MzAwNDcxMjI2MA==&mid=2247484455&idx=1&sn=e1b4324ddcf7d6123be30d9a5613e17b&chksm=9b26f60cac517f1a920cf3b73b3212a645aeef78882c47957b9f3c2135cb7ce051c73fe77bb2&mpshare=1&scene=23&srcid=1111auAYWmr1N0NAs9Wp2hGz&sharer_sharetime=1605145141579&sharer_shareid=5051b3eddbbe2cb698aedf9452370026#rd

CTF中常见密码题解密网站总结（建议收藏）

https://blog.csdn.net/qq_41638851/article/details/100526839

CTF密码学常见加密解密总结（建议收藏）

https://blog.csdn.net/qq_40837276/article/details/83080460

## 代码

1、密码存储（后端处理）
X3.2-md5&salt

```
function add_user() {
$password = md5(md5($password).$salt);
}
```

2 python md5加密脚本

```
import hashlib
text = "123456"
md5_obj = hashlib.md5()
md5_obj.update(text.encode('utf-8'))
print(md5_obj.hexdigest())
```

3 博客登录-混合加密（前端处理）

```
<script src="/Scripts/Vip/Login.js?v=20241202154949"></script>

function Login() {

logindata.UserName = encodeURI(encrypt.encrypt(numMobile));

logindata.Mobile = encodeURI(encrypt.encrypt(numMobile));;

logindata.Password = encodeURI(encrypt.encrypt(numPassword));

}

var encrypt = new JSEncrypt();

encodeURI(encrypt.encrypt('13554365566'));
```

## 密文

明确以下三种加密算法加解密条件

对称AES/DES加解密，非对称RSA加解密

解密：http://tool.chacuo.net/cryptdes

## 应用场景

1、发送数据的时候自动将数据加密发送（自需加密即可）

安全测试思路：我们需要将我们的Payload也要加密发送过去，这样才符合正常的业务逻辑，所以我们就只需要调用应用的JS加密逻辑进行提交发送测试即可！

2、比如要得到数据的明文（必须要拿到解密算法）

由于各种算法的解密条件不一，密钥，偏移量，私钥等不一定能拿到。

## 影响

1、目标：API接口 前后端分离应用居多

2、漏洞发现：未授权 各类漏洞测试 爆破等

3、其他