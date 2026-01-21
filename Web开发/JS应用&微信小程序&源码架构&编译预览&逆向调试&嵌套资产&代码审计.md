创建小程序模版引用

https://developers.weixin.qq.com/miniprogram/dev/devtools/download.html

## 小程序架构

1、主体结构

小程序包含一个描述整体程序的 app 和多个描述各自页面的 page。
一个小程序主体部分(即app)由三个文件组成，必须放在项目的根目录，如下：

文件 必需 作用

app.js 是 小程序逻辑

app.json 是 小程序公共配置

app.wxss 否 小程序公共样式表

2、一个小程序页面由四个文件组成，分别是:

xxx.js 页面逻辑

xxx.json 页面配置

xxx.wxml 页面结构

xxx.wxss 页面样式

3、项目整体目录结构

pages 页面文件夹

index 首页

logs 日志

utils

util 工具类(mina框架自动生成,你也可建立：api)

app.js 入口js(类似于java类中的main方法)、全局js

app.json 全局配置文件

app.wxss 全局样式文件

project.config.json 跟你在详情中勾选的配置一样

sitemap.json 用来配置小程序及其页面是否允许被微信索引

## 小程序开发

1、可视化

2、真机调试

3、编译预览

4、NPM编译

案例1：嵌套Web应用资产

```
<web-view src="http://www.xiaodi8.com"></web-view>
```
案例2：嵌套第三方云服务

阿里云OSS存储-AK/SK

## 小程序安全

1、逆向反编译

2、敏感信息泄露（AK/SK,APIKEY等）

3、资产信息提取（IP,Web应用等）

4、代码逻辑安全（算法，提交接口等）

## 案例

https://mp.weixin.qq.com/s/z28ppqhNJnLVWSScMEqiuw

https://mp.weixin.qq.com/s/ZfovaAyipqzUIYdL9objPA

https://mp.weixin.qq.com/s/PK1NhvdrDr3XWEliuyEiig