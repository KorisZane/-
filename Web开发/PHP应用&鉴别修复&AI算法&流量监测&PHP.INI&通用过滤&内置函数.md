## php.ini设置

https://www.yisu.com/ask/28100386.html

https://blog.csdn.net/u014265398/article/details/109700309

https://www.cnblogs.com/xiaochaohuashengmi/archive/2011/10/23/2222105.html

安全模式 safe_mode 命令执行函数会被禁用

*路径访问 open_basedir 限制文件操作安全（遍历等）

*禁用函数 disable_function 升级版安全模式，自定义限制函数

*魔术引号转义 magic_quotes_gpc 同理下面的sql过滤第一个函数

数据库访问次数 max_connections 防止数据库爆破

禁用远程执行 allow_url_include allow_url_fopen远程包含开关等

*安全会话管理 session.cookie_httponly session.cookie_secure

防止跨站脚本攻击（XSS）和中间人攻击（MITM）

## 引用全局文件

1、关键内容检测（黑白名单）

文件上传，SQL注入，XSS跨站等

演示：文件上传，SQL注入，XSS跨站

2、模仿流量检测（基于规则或AI算法）

演示：Python Flask+PHP Curl+训练大模型

客户端请求数据 -> 中间件搭建平台 -> 服务器代码文件处理

客户端请求数据 -> WAF或流量监控 （规则，AI模型算法）-> 正常数据 -> 中间件搭建平台 -> 服务器代码文件处理

客户端请求数据 -> WAF或流量监控 （规则，AI模型算法）-> 异常数据 -> 截止

## 代码-内置函数

检测：数据的类型差异，数据的固定内容

gettype()获取变量的类型

is_float()检测变量是否是浮点型

is_bool()检测变量是否是布尔型

is_int()检测变量是否是整数

is_null()检测变量是否为NULL

is_numeric()检测变量是否为数字或数字字符串

is_object()检测变量是否是一个对象

is_resource()检测变量是否为资源类型

is_scalar()检测变量是否是一个标量

is_string()检测变量是否是字符串

is_array()检测变量是否是数组

filter_var()使用特定的过滤器过滤一个变量

FILTER_SANITIZE_STRING 过滤器可以过滤HTML标签和特殊字符

FILTER_SANITIZE_NUMBER_INT 过滤器可过滤非整数字符

FILTER_SANITIZE_URL 过滤器用于过滤URL中的非法字符

FILTER_VALIDATE_EMAIL 过滤器来验证电子邮件地址的有效性

演示：filter_var() is_numeric() is_string()等

### sql注入过滤

Addslashes()返回字符串，该字符串为了数据库查询语句等的需要在某些字符前加上了反斜线。这些字符是单引号()、双引号(”)、反斜线()与NULL字符)。

stripslashes()反引用一个引用字符串,如果magic_quotes_sybase项开启，反斜线将被去除，但是两个反斜线将会被替换成一个。

addcslashes()返回字符串，该字符串在属于参数charlist列表中的字符前都加上了反斜线。

stripcslashes()返回反转义后的字符串。可识别类似C语言的\n，r，…八进制以及十六进制的描述。

mysql_escape_string()此函数并不转义%和_。作用和mysql real escape_string()基本一样

mysql_real_escape_string()调用mysql库的函数在以下字符前添加反斜杠:x00、\n、\r、\、x1a

PHP魔术引号当打开时，所有的'(单引号)，”(双引号)，(反斜线)和NULL字符都会被自动加上一个反斜线进行转义。这和addslashes()作用完全相同。

预编译机制

演示：魔术引号，addslashes()，预编译等

### XSS跨站过滤

htmlspecialchars()函数把预定义的字符转换为HTML实体。

strip_tags()函数剥去字符串中的HTML、XML以及PHP的标签。

演示：htmlspecialchars() strip_tags()

### 命令执行过滤

escapeshellcmd()确保用户只执行一个命令用户可以指定不限数量的参数用户不能执行不同的命令

escapeshellarg()确保用户只传递一个参数给命令用户不能指定更多的参数一个用户不能执行不同的命令

## 其他漏洞过滤

可参考AI答案或参数类型过滤等