## 相关靶场

https://github.com/bewhale/JavaSec
https://github.com/whgojp/JavaSecLab
https://github.com/j3ers3/Hello-Java-Sec

## SQL注入

- JDBC

1. 采用Statement方法拼接SQL语句

```
// 原生sql语句动态拼接 参数未进行任何处理
public R vul1(String type,String id,String username,String password) {
    //注册数据库驱动类
    Class.forName("com.mysql.cj.jdbc.Driver");

    //调用DriverManager.getConnection()方法创建Connection连接到数据库
    Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPass);

    //调用Connection的createStatement()或prepareStatement()方法 创建Statement对象
    Statement stmt = conn.createStatement();
    switch (type) {
        case "add":
            //这里没有标识id id自增长
            sql = "INSERT INTO sqli (username, password) VALUES ('" + username + "', '" + password + "')";
            //通过Statement对象执行SQL语句，得到ResultSet对象-查询结果集
            // 这里注意一下 insert、update、delete 语句应使用executeUpdate()
            rowsAffected = stmt.executeUpdate(sql);
            //关闭ResultSet结果集 Statement对象 以及数据库Connection对象 释放资源
            stmt.close();
            conn.close();
            return R.ok(message);
        case "delete":
            sql = "DELETE FROM users WHERE id = '" + id + "'";
            rowsAffected = stmt.executeUpdate(sql);
            ...
        case "update":
            sql = "UPDATE sqli SET password = '" + password + "', username = '" + username + "' WHERE id = '" + id + "'";
            rowsAffected = stmt.executeUpdate(sql);
            ...
        case "select":
            sql = "SELECT * FROM users WHERE id  = " + id;
            ResultSet rs = stmt.executeQuery(sql);
            ...
        }
}
```

2. PrepareStatement会对SQL语句进行预编译，但如果直接采取拼接的方式构造SQL，此时进行预编译也无用。

```
// 虽然使用了conn.prepareStatement(sql)创建了一个PreparedStatement对象，但在执行 stmt.executeUpdate(sql)时，却是传递了完整的SQL语句作为参数，而不是使用了预编译的功能
public R vul2(String type,String id,String username,String password) {
    Class.forName("com.mysql.cj.jdbc.Driver");
    Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPass);
    PreparedStatement stmt;
    switch (type) {
        case "add":
            sql = "INSERT INTO sqli (username, password) VALUES ('" + username + "', '" + password + "')";
            stmt = conn.prepareStatement(sql);
            rowsAffected = stmt.executeUpdate(sql);
            ...
        case "delete":
            sql = "DELETE FROM users WHERE id = '" + id + "'";
            stmt = conn.prepareStatement(sql);
            rowsAffected = stmt.executeUpdate(sql);
            ...
        case "update":
            sql = "UPDATE sqli SET username = '" + username + "', password = '" + password + "' WHERE id = '" + id + "'";
            stmt = conn.prepareStatement(sql);
            rowsAffected = stmt.executeUpdate(sql);
            ...
        case "select":
            sql = "SELECT * FROM users WHERE id  = " + id;
            stmt = conn.prepareStatement(sql);
            ResultSet rs = stmt.executeQuery(sql);
            ...
    }
}
```

3. JDBCTemplate是Spring对JDBC的封装，如果使用拼接语句便会产生注入
4. 自定义过滤（黑白名单）

	*安全写法：SQL语句占位符（?） + PrepareStatement预编译*

```
// 采用预编译的方法，使用?占位，也叫参数化的SQL
public R safe1(String type,String id,String username,String password) {
    Class.forName("com.mysql.cj.jdbc.Driver");
    Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPass);
    PreparedStatement stmt;
    switch (type) {
        case "add":
            // 这里可以看到使用了?占位符 sql语句和参数进行分离
            sql = "INSERT INTO users (username, password) VALUES (?, ?)"; 
            stmt = conn.prepareStatement(sql);
            // 参数化处理
            stmt.setString(ueditor, username); 
            stmt.setString(2, password);
            // 使用预编译时 不需要传递sql语句
            rowsAffected = stmt.executeUpdate();
        case "delete":
            sql = "DELETE FROM users WHERE id = ?";
            stmt = conn.prepareStatement(sql);
            stmt.setString(ueditor, id);
            rowsAffected = stmt.executeUpdate();
            ...
        case "update":
            sql = "UPDATE sqli SET username = ?, password = ? WHERE id = ?";
            stmt = conn.prepareStatement(sql);
            stmt.setString(1, username);  
            stmt.setString(2, password);
            stmt.setString(3, id);
            stmt.executeUpdate();
            ...
        case "select":
            sql = "SELECT * FROM users WHERE id  = ?";
            stmt = conn.prepareStatement(sql);
            stmt.setString(ueditor, id);
            ResultSet rs = stmt.executeQuery();
            ...
   }
}
```

 Web安全框架-采用ESAPI过滤

```
// ESAPI提供了多种输入验证API，提供对XSS攻击和SQL注入攻击等的防护
public R safe4(String id) {
    Codec<Character> oracleCodec = new OracleCodec();
    Class.forName("com.mysql.cj.jdbc.Driver");
    Connection conn = DriverManager.getConnection(dbUrl, dbUser, dbPass);

    Statement stmt = conn.createStatement();
    // 使用了 Oracle 的编解码器 OracleCodec 和 ESAPI 库来对 ID 进行编码，以防止 SQL 注入攻击。
    String sql = "select * from sqli where id = '" + ESAPI.encoder().encodeForSQL(oracleCodec, id) + "'";
    // String sql = "select * from sqli where id = '" + id + "'";
    String sql = "select * from users where id = '" + id + "'";
    ResultSet rs = stmt.executeQuery(sql);
}
```

- MyBatis
```
MyBatis支持两种参数符号，一种是#，另一种是$，#使用预编译，$使用拼接SQL
```

1. order by注入：由于使用#{}会将对象转成字符串，形成order by "user" desc造成错误，因此很多研发会采用${}来解决，从而造成注入.

2. like 注入：模糊搜索时，直接使用'%#{q}%' 会报错，部分研发图方便直接改成'%${q}%'从而造成注入.

3. in注入：in之后多个id查询时使用#同样会报错，从而造成注入

看Mapper层语句

order by演示
```
// Controller层
public R special1OrderBy() {
  List<Sqli> sqlis = new ArrayList<>();
  switch (type) {
      case "raw":
          sqlis = sqliService.orderByVul(field);
          break;
      case "prepareStatement":
          sqlis = sqliService.orderByPrepareStatement(field);
          break;
      case "writeList":
          sqlis = sqliService.orderByWriteList(field);
      ...
// Service层
//自定义SQL-使用#{}
@Override
public List<Sqli> orderByVul(String field) {
    return sqliMapper.orderByVul(field);
}
@Override
public List<Sqli> orderByPrepareStatement(String field) {
    return sqliMapper.orderByPrepareStatement(field);
}
@Override
public List<Sqli> orderByWriteList(String field) {
    return sqliMapper.orderByWriteList(field);
}
// Mapper层
<!--    Order by下的${}拼接注入问题-->
<select id="orderByVul" resultType="top.whgojp.modules.sqli.entity.Sqli">
    SELECT * FROM sqli
    <if test="field != null and field != ''">
        ORDER BY ${field}
    </if>
</select>
<!--    Order by下的#{}写法 排序不生效-->
<select id="orderByPrepareStatement" resultType="top.whgojp.modules.sqli.entity.Sqli">
    SELECT * FROM sqli
    <if test="field != null and field != ''">
        ORDER BY #{field}
    </if>
</select>
<!--    Order by下的安全写法 白名单-->
<select id="orderByWriteList" resultType="top.whgojp.modules.sqli.entity.Sqli">
    SELECT * FROM sqli
    <if test="field != null and field != ''">
        <choose>
            <!-- 排序列名白名单 -->
            <when test="field == 'id' or field == 'username' or field == 'password'">
                ORDER BY ${field}
            </when>
            <otherwise>
                <!-- 默认使用id进行排序 -->
                ORDER BY id
            </otherwise>
        </choose>
    </if>
</select>
```

- Hibernate

1. setParameter：预编译

2. username=:username 预编译

缺陷代码

```
public R vul1(@RequestParam String username) {
    try {
        String sql = "SELECT * FROM sqli WHERE username = '" + username + "'";
        Object[] result = (Object[]) hibernateTemplate.execute(session ->
                session.createNativeQuery(sql).uniqueResult()
        );
        message = "查询成功，用户名：" + result[1] + " 密码：" +result[2];
        return R.ok(message);
    } catch (Exception e) {
        log.error("查询失败", e);
        return R.error(e.getMessage());
    }
}

public R vul2(@RequestParam String username) {
    try {
        String hql = "FROM Sqli WHERE username = '" + username + "'";
        Sqli result = (Sqli) hibernateTemplate.execute(session ->
                session.createQuery(hql).uniqueResult()
        );
        message = "查询成功，用户名：" +result.getUsername()+ " 密码：" +result.getPassword();
        return R.ok(message);
    } catch (Exception e) {
        log.error("查询失败", e);
        return R.error(e.getMessage());
    }
}
```

安全代码

```
public R safe(@RequestParam String username) {
    try {
        String hql = "FROM Sqli WHERE username = :username";
        Sqli result = hibernateTemplate.execute(session ->
                (Sqli) session.createQuery(hql)
                        .setParameter("username", username)
                        .uniqueResult()
        );
        message = "查询成功，用户名：" +result.getUsername()+ " 密码：" +result.getPassword();
        return R.ok(message);
    } catch (Exception e) {
        log.error("查询失败", e);
        return R.error(e.getMessage());
    }
}
```

## XXE注入

代码审计点

```
XMLReader
SAXReader
DocumentBuilder
XMLStreamReader
SAXBuilder
SAXParser
SAXSource
TransformerFactory
SAXTransformerFactory
SchemaFactory
Unmarshaller
XPathExpression
```

缺陷代码

```
public String vul1(String payload) {
    try {
        XMLReader xmlReader = XMLReaderFactory.createXMLReader();
        StringWriter stringWriter = new StringWriter();
        xmlReader.setContentHandler(new DefaultHandler() {
            public void characters(char[] ch, int start, int length) {
                for (int i = start; i < start + length; i++) {
                    if (ch[i] == '\n') {
                        stringWriter.write("<br/>");
                    } else {
                        stringWriter.write(ch[i]);
                    }
                }
            }
        });
        xmlReader.parse(new InputSource(new StringReader(payload)));
        return stringWriter.toString();
    } catch (Exception e) {
        return e.getMessage();
    }
}


public String vul2(String payload) {
    try {
        SAXParserFactory factory = SAXParserFactory.newInstance();
        SAXParser parser = factory.newSAXParser();
        ...
        parser.parse(new InputSource(new StringReader(payload)), handler);
        return stringWriter.toString();
    } catch (Exception e) {
        return e.toString();
    }
}
```

*获取适用以上12种类函数实现，parse后续的可控变量*

## RCE

四个类造成RCE

1、ProcessBuilder

2、Runtime.getRuntime().exec()

3、ProcessImpl

4、GroovyShell

## SSRF

php造成SSRF漏洞函数file_get_content()

URL类的引用

有可控变量造成SSRF

缺陷代码

```
public String vul(String url) {
    try {
        URL u = new URL(url);
        // 这里以URLConnection作为演示
        URLConnection conn = u.openConnection();
        BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
        String content;
        StringBuilder html = new StringBuilder();
        html.append("<pre>");
        while ((content = reader.readLine()) != null) {
            html.append(content).append("\n");
        }
        html.append("</pre>");
        reader.close();
        return html.toString();
    } catch (Exception e) {
        return e.getMessage();
    }
}
```

## URL跳转

1、Spring MVC-redirect ModelAndView

```
// 基于Spring MVC的重定向方式
// 通过返回带有 redirect: 前缀的字符串来实现重定向。
public String vul1(@RequestParam("url") String url) {
    return "redirect:" + url;   // Spring MVC写法 302临时重定向
}

// 通过返回 ModelAndView 对象并指定 redirect: 前缀来实现重定向。
public ModelAndView vul2(@RequestParam("url") String url) {
    return new ModelAndView("redirect:" + url); // Spring MVC写法 使用ModelAndView 302临时重定向
}
```

2、HttpServlet-setHeader sendRedirect

```
// 基于Servlet标准的重定向方式
// 通过设置响应状态码和头部信息实现重定向。
public static void vul2(HttpServletRequest request, HttpServletResponse response) {
    String url = request.getParameter("url");
    response.setStatus(HttpServletResponse.SC_MOVED_PERMANENTLY); // 301永久重定向
    response.setHeader("Location", url);
}

// 通过调用 HttpServletResponse.sendRedirect() 实现重定向。
public static void vul3(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String url = request.getParameter("url");
    response.sendRedirect(url); // 302临时重定向
}
```

3、Spring-ResponseEntity setHeader

```
// 基于Spring注解和状态码的重定向方式
// 使用ResponseEntity设置状态码实现重定向
public ResponseEntity<Void> vul5(@RequestParam("url") String url) {
    HttpHeaders headers = new HttpHeaders();
    headers.setLocation(URI.create(url));
    return new ResponseEntity<>(headers, HttpStatus.FOUND); // 302临时重定向
}

// 通过注解设置状态码实现重定向
@ResponseStatus(HttpStatus.FOUND) // 302临时重定向
public void vul6(HttpServletRequest request, HttpServletResponse response) throws IOException {
    String url = request.getParameter("url");
    response.setHeader("Location", url);
}
```

总结：关注什么技术栈实现源码，看类函数触发可控变量