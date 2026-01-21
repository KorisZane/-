## 公众号资产

公众号资产就是找外部链接，转Web测试

## 供应链

这是一种典型的迂回攻击方式。攻击者将目光聚集在目标企业的上下游供应商，比如IT供应商、安全供应商等，从这些上下游企业中找到软件或系统、管理上的漏洞，进而攻进目标企业内部。

-商业购买系统

-软件开发商

-外包业务

-代理商

-招投标文件

### 思路

1、找供应链公司，通过演示站点或提供的ico或特征文件找到大量高校使用系统的目标，一旦测出这个应用产品系统的漏洞，形成了通杀

2、找高校目标，通过高校系统的信息或者ico等找到供应链，再从供应链继续找使用同样的系统的高校目标，一旦测出这个应用产品系统的漏洞，形成了通杀

### 上游供应链打下游

知道软件开发厂商，找使用产品的甲方

### 下游供应链打上游

找相似目标存在其他脆弱点

### 找系统漏洞

1、尽量找到源码（源码获取途径）

2、旁入方法（找相似目标的存在脆弱点为突破）

### 案例

https://mp.weixin.qq.com/s/YSD6yGGlBchY61hAZdKLoA

https://mp.weixin.qq.com/s/a8wyCeY7j29j6ITXfisOtQ

https://mp.weixin.qq.com/s/LQ5yeVJK4yCysxArwxSYbg

## 信息泄漏

1、Github监控

GITHUB资源搜索：

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

***什么白嫖各种api key了 会员账号密码了***