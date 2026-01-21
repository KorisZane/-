## 前端加密

常见的加密

```
<script src="crypto-js.js"></script>  
<script>  
  var str="qiyi123456"  
  
  //Base64  
  var base64str = window.btoa(str)  
  console.log(base64str)  
  
  //MD5  
  var md5str = CryptoJS.MD5(base64str).toString()  
  console.log(md5str)  
  
  //SHA  
  var shastr = CryptoJS.SHA1(str).toString();  
  console.log(shastr)  
  
  //HMAC  
  var key1 = "qqqqq";  
  var hash = CryptoJS.HmacSHA256(key1,str);  
  var hmacstr = CryptoJS.enc.Hex.stringify(hash);  
  console.log(hmacstr)  
  
  //AES  
  var key2="wwwwwwwwwwwwwwww";  
  var aesstr = CryptoJS.AES.encrypt(str, CryptoJS.enc.Utf8.parse(key2),  
          {  
            mode: CryptoJS.mode.ECB,  
            padding: CryptoJS.pad.Pkcs7  
          }  
  ).toString();  
  console.log(aesstr)  
  
  //DES  
  var key3 = "wwwwwwwwwwwwwwww"  
  var desstr = CryptoJS.DES.encrypt(str, CryptoJS.enc.Utf8.parse(key3),  
          {  
            mode: CryptoJS.mode.ECB,  
            padding: CryptoJS.pad.Pkcs7  
          }).toString()  
  console.log(desstr)  
</script>  
  
<script src="jsencrypt.js"></script>  
<script>  
  var str="12345"  
  //RSA加密  
  var PUBLIC_KEY = '-----BEGIN RSA PUBLIC KEY-----\n' +  
          'MIIBCgKCAQEA2XCLPSpwwbhanLeYPysj7JCm338gpfIaptmGRL38S3j9xCBxSjkI\n' +  
          'WD8Xsjs9O11AEjSvigQpWezY3H+SScNoBUN5cdz/oPU5sMvvzLRXRT3pUfsY2527\n' +  
          'fMymZBStsrEMnjswjJyaDkSpvMV7s7aCXPk2AGnWpqPlLkByWGzhNbc1U/i9aSH3\n' +  
          'O7lc2MmDiUo2siupPDkndBP7BC3AMeUgLPd4RVJTmN0/9jNLhJ/1biHYdZ5Ays+S\n' +  
          'TYcZ9JX2PDdZB8WpGxj3IT3wCJd06zXi4MGqSdiyubx6z6Ejhy9xR8L9YERY5R8p\n' +  
          'QyiVAck+hASpPYAbptI26fjEXFXNMi3tKQIDAQAB\n' +  
          '-----END RSA PUBLIC KEY-----\n'  
  var encrypt = new JSEncrypt();  
  encrypt.setPublicKey(PUBLIC_KEY)  
  var rsastr = encrypt.encrypt(str);  
  console.log(rsastr)  
  
</script>
```

前端加密的数据需要js逆向，找出加密方式，密钥key等，需要发送与前端加密相同的数据后端才能正常接收参数

***前端加密一般不用RSA我猜测是因为RSA对数据长度有限制***

## 实际案例

[[JS+代码混淆.pdf]]