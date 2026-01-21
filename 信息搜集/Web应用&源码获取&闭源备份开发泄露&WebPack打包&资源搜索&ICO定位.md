参考
https://www.secpulse.com/archives/124398.html

https://mp.weixin.qq.com/s/QgLDdaefXlZtvlSiFQShZw

https://mp.weixin.qq.com/s/zhjxW2mUEgq6dFAp3kBcgQ

## 泄露原因

1、从源码本身的特性入口

2、从管理员不好的习惯入口

3、从管理员习惯的开发入口

4、从管理员不好的配置入口

5、从管理员不好的意识入口

6、从管理员资源信息搜集入口

## 泄露合集

Webpack打包泄漏

composer.json

GitHub源码泄漏

git源码泄露

svn源码泄露

网站备份压缩文件

DS_Store文件泄露

hg源码泄漏

SWP 文件泄露

CVS泄露

Bzr泄露

WEB-INF/web.xml 泄露

## 思路点

1、使用指纹识别平台获取到目标是什么名字的源码程序

通过搜索引擎获取到源码

2、使用指纹识别平台不能获取目标信息

后端：借助svn git composer.json ds_store 备份扫描等配置泄漏安全

前端：webpack打包 谷歌插件获取源码

如果没有这些问题 还想获取源码怎么办 见下面方法

3、借助资源平台去搜索（github gitee oschina等源码）

-特定文件

-特征联系方式

-github监控（ 除了源码，漏洞更新，工具更新等都有用处）
目标使用的源码是开源还是闭源，是否公开

## 演示案例

1、已知指纹识别获取源码途径

2、GIT泄漏-未知指纹识别获取源码途径

3、SVN泄漏-未知指纹识别获取源码途径

4、DS_Store泄漏-未知指纹识别获取源码途径

GIT：https://github.com/lijiejie/GitHack

SVN：https://github.com/callmefeifei/SvnHack

DS_Store：https://github.com/lijiejie/ds_store_exp

通用：https://github.com/0xHJK/dumpall

5、WebPack打包-未知指纹识别获取源码途径

https://github.com/NothingCw/SourceDetector-dist

6、composer.json-未知指纹识别获取源码途径

7、源码文件备份泄漏-未知指纹识别获取源码途径

目录文件扫描工具获取存在，直接访问

8、Github资源搜索-未知指纹识别获取源码途径

9、另类目标源码获取

https://www.huzhan.com/

https://www.goofish.com/

从目标转到寻找源码系统上，从源码系统上找应用目标

## 拓展

资源搜索语法：

in:name test #仓库标题搜索含有关键字

in:descripton test #仓库描述搜索含有关键字

in:readme test #Readme文件搜素含有关键字

stars:>3000 test #stars数量大于3000的搜索关键字

stars:1000..3000 test #stars数量大于1000小于3000的搜索关键字 forks:>1000 test #forks数量大于1000的搜索关键字

forks:1000..3000 test #forks数量大于1000小于3000的搜索关键字 size:>=5000 test #指定仓库大于5000k(5M)的搜索关键字 pushed:>2019-02-12 test #发布时间大于2019-02-12的搜索关键字 created:>2019-02-12 test #创建时间大于2019-02-12的搜索关键字 user:test #用户名搜素

license:apache-2.0 test #明确仓库的 LICENSE 搜索关键字 language:java test #在java语言的代码中搜索关键字

user:test in:name test #组合搜索,用户名test的标题含有test的

关键字配合谷歌搜索：

site:Github.com smtp

site:Github.com smtp @qq.com

site:Github.com smtp @126.com

site:Github.com smtp @163.com

site:Github.com smtp @sina.com.cn

site:Github.com smtp password

site:Github.com String password smtp

## 案例文章

渗透测试实战---某资金盘信息收集(.idea目录泄露)

https://mp.weixin.qq.com/s/7cSrDZci_drE6wTGvQOOHw

SRC挖掘-js.map泄露到接管云上域控

https://mp.weixin.qq.com/s/QZsNjbTiaqC1qwzARVAiGQ