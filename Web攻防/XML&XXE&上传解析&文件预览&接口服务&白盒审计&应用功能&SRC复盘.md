## 黑盒功能案例

1、不安全的图像读取-SVG

2、不安全的文档转换-DOCX

3、不安全的传递服务-SOAP

## 白盒功能案例

1、漏洞函数simplexml_load_string

2、pe_getxml函数调用了漏洞函数

3、wechat_getxml调用了pe_getxml

4、notify_url调用了wechat_getxml

访问notify_url文件触发wechat_getxml函数,构造Paylod测试

技术文章
https://xz.aliyun.com/news/16463

## 实际复盘

https://mp.weixin.qq.com/s/biQgwMU2v1I92CsDOFRB7g
https://mp.weixin.qq.com/s/1pj9sbwKT6RjIiLgNC7-Gg
https://mp.weixin.qq.com/s/Mgd91_Iie-wZU7MqP5oCXw