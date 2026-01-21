## 文件下载

常规下载URL：http://www.xiaodi8.com/upload/123.pdf
可能存在安全URL：http://www.xiaodi8.com/xx.xx?file=123.pdf

下载敏感文件，如包含数据库操作的文件

利用：常规下载敏感文件（数据库配置，中间件配置，系统密匙等文件信息）

## 文件删除

可能存在安全问题：前台或后台有删除功能应用
利用：常规删除重装锁定配合程序重装或高危操作

## 目录索引

1、目录索引

目录权限控制不当，通过遍历获取到有价值的信息文件去利用

2、目录穿越（路径遍历常出现后台中）

目录权限控制不当，通过控制查看目录路径穿越到其他目录或判断获取价值文件再利用

## 黑盒分析

1、功能点

文件上传，文件下载，文件删除，文件管理器等地方

2、URL特征

文件名：

download，down，readfile，read，del，dir，path，src，Lang等

参数名：

file、path、data、filepath、readfile、data、url、realpath等

## 白盒分析

上传类，删除类，下载类，目录操作函数，读取查看方法或函数等

## SRC报告

https://mp.weixin.qq.com/s/Kiec7FhvpAmPf7zTWvJa3g
https://mp.weixin.qq.com/s/QqXxpTwXSjCNN622fSsZAQ
https://mp.weixin.qq.com/s/A2FvZMuPpHiewGrL5UDlHw
https://mp.weixin.qq.com/s/w2PH7EI6Z2R9diV_tBA1Zw

