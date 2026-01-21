## Maven配置

https://blog.csdn.net/cxy2002cxy/article/details/144809310

## JDBC

参考 https://www.jianshu.com/p/ed1a59750127

### 引用依赖(pom.xml)

```
<?xml version="1.0" encoding="UTF-8"?>  
<project xmlns="http://maven.apache.org/POM/4.0.0"  
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"  
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">  
    <modelVersion>4.0.0</modelVersion>  
  
    <groupId>com.example</groupId>  
    <artifactId>Jdec-Demo</artifactId>  
    <version>1.0-SNAPSHOT</version>  
    <name>Jdec-Demo</name>  
    <packaging>war</packaging>  
  
    <properties>        <project.build.sourceEncoding>UTF-8</project.build.sourceEncoding>  
        <maven.compiler.target>1.8</maven.compiler.target>  
        <maven.compiler.source>1.8</maven.compiler.source>  
        <junit.version>5.10.2</junit.version>  
    </properties>  
    <dependencies>        <!-- https://mvnrepository.com/artifact/mysql/mysql-connector-java -->  
        <dependency>  
            <groupId>mysql</groupId>  
            <artifactId>mysql-connector-java</artifactId>  
            <version>5.1.48</version>  
        </dependency>        <dependency>            <groupId>javax.servlet</groupId>  
            <artifactId>javax.servlet-api</artifactId>  
            <version>4.0.1</version>  
            <scope>provided</scope>  
        </dependency>        <dependency>            <groupId>org.junit.jupiter</groupId>  
            <artifactId>junit-jupiter-api</artifactId>  
            <version>${junit.version}</version>  
            <scope>test</scope>  
        </dependency>        <dependency>            <groupId>org.junit.jupiter</groupId>  
            <artifactId>junit-jupiter-engine</artifactId>  
            <version>${junit.version}</version>  
            <scope>test</scope>  
        </dependency>    </dependencies>  
    <build>        <plugins>            <plugin>                <groupId>org.apache.maven.plugins</groupId>  
                <artifactId>maven-war-plugin</artifactId>  
                <version>3.3.2</version>  
            </plugin>        </plugins>    </build></project>
```

### 不安全写法

JdbcServlet.java(拼接写法)

```
package com.example.jdecdemo.Servlet;  
  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;  
import java.sql.*;  
  
@WebServlet(name = "jdbc", value = "/sql")  
public class JdbcServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        String id = req.getParameter("id");  
        String sql = "select * from admin where id=" + id;  
        String url = "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8";  
        try {  
            Class.forName("com.mysql.jdbc.Driver");  
            Connection connection = DriverManager.getConnection(url, "root", "root");  
            //System.out.println(connection);  
            Statement statement = connection.createStatement();  
            ResultSet resultSet = statement.executeQuery(sql);  
            //System.out.println(resultSet);  
            while (resultSet.next()) {  
                resp.getWriter().println(resultSet.getString("id"));  
                resp.getWriter().println(resultSet.getString("username"));  
                resp.getWriter().println(resultSet.getString("password"));  
                resp.getWriter().println(resultSet.getString("creaded_at"));  
            }  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

### 安全写法(预编译)

```
package com.example.jdecdemo.Servlet;  
  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.HttpServlet;  
import javax.servlet.http.HttpServletRequest;  
import javax.servlet.http.HttpServletResponse;  
import java.io.IOException;  
import java.sql.*;  
  
@WebServlet(name = "jdbc", value = "/sql")  
public class JdbcServlet extends HttpServlet {  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {  
        String id = req.getParameter("id");  
        String sql = "select * from admin where id=?";  
        String url = "jdbc:mysql://localhost:3306/test?useUnicode=true&characterEncoding=utf-8";  
        try {  
            Class.forName("com.mysql.jdbc.Driver");  
            Connection connection = DriverManager.getConnection(url, "root", "root");  
            PreparedStatement preparedStatement = connection.prepareStatement(sql);  
            preparedStatement.setString(1, id);  
            ResultSet resultSet = preparedStatement.executeQuery();  
            while (resultSet.next()) {  
                resp.getWriter().println(resultSet.getString("id"));  
                resp.getWriter().println(resultSet.getString("username"));  
                resp.getWriter().println(resultSet.getString("password"));  
                resp.getWriter().println(resultSet.getString("creaded_at"));  
            }  
        } catch (ClassNotFoundException e) {  
            throw new RuntimeException(e);  
        } catch (SQLException e) {  
            throw new RuntimeException(e);  
        }  
    }  
}
```

## Mybatis

### 项目结构

```
mybatis-sqlinjection-demo
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── demo
        │           ├── entity
        │           │   └── User.java
        │           ├── mapper
        │           │   └── UserMapper.java
        │           └── servlet
        │               └── LoginServlet.java
        │
        ├── resources
        │   ├── mybatis-config.xml
        │   └── mapper
        │       └── UserMapper.xml
        │
        └── webapp
            ├── WEB-INF
            │   └── web.xml   （可选，使用 @WebServlet 可不要）
            └── index.jsp
```

### mybatis-config.xml

```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE configuration  
        PUBLIC "-//mybatis.org//DTD Config 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-config.dtd">  
  
<configuration>  
  
    <environments default="dev">  
        <environment id="dev">  
            <transactionManager type="JDBC"/>  
            <dataSource type="POOLED">  
                <property name="driver" value="com.mysql.cj.jdbc.Driver"/>  
                <property name="url" value="jdbc:mysql://localhost:3306/mybatis_test?useSSL=false&amp;serverTimezone=UTC"/>  
                <property name="username" value="root"/>  
                <property name="password" value="root"/>  
            </dataSource>        </environment>    </environments>  
    <mappers>        <mapper resource="UserMapper.xml"/>  
    </mappers>  
</configuration>
```

### UserMapper.cml

```
<?xml version="1.0" encoding="UTF-8" ?>  
<!DOCTYPE mapper  
        PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN"  
        "http://mybatis.org/dtd/mybatis-3-mapper.dtd">  
  
<mapper namespace="com.example.mygbatisdemo.mapper.UserMapper">  
  
    <!-- ❌ SQL 注入漏洞点 -->  
    <select id="findUserByName" resultType="User">  
        SELECT id, username, password  
        FROM users        WHERE username = '${username}'    </select>  
  
</mapper>
```

### User.java

```
package com.example.mygbatisdemo.entity;  
  
public class User {  
    private int id;  
    private String username;  
    private String password;  
  
    // getter / setter  
    public int getId() { return id; }  
    public void setId(int id) { this.id = id; }  
  
    public String getUsername() { return username; }  
    public void setUsername(String username) { this.username = username; }  
  
    public String getPassword() { return password; }  
    public void setPassword(String password) { this.password = password; }  
}
```

### UserMapper

```
package com.example.mygbatisdemo.mapper;  
  
import com.example.mygbatisdemo.entity.User;  
  
import java.util.List;  
  
public interface UserMapper {  
    // ⚠️ 不安全：参数直接拼 SQL    List<User> findUserByName(String username);  
}
```

### LoginServlet.java

```
package com.example.mygbatisdemo.servlet;  
  
import com.example.mygbatisdemo.entity.User;  
import com.example.mygbatisdemo.mapper.UserMapper;  
import org.apache.ibatis.io.Resources;  
import org.apache.ibatis.session.*;  
  
import javax.servlet.ServletException;  
import javax.servlet.annotation.WebServlet;  
import javax.servlet.http.*;  
import java.io.IOException;  
import java.io.PrintWriter;  
import java.io.Reader;  
import java.util.List;  
  
@WebServlet("/login")  
public class LoginServlet extends HttpServlet {  
  
    private SqlSessionFactory factory;  
  
    @Override  
    public void init() throws ServletException {  
        try {  
            Reader reader = Resources.getResourceAsReader("mybatis-config.xml");  
            factory = new SqlSessionFactoryBuilder().build(reader);  
        } catch (Exception e) {  
            throw new ServletException(e);  
        }  
    }  
  
    @Override  
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)  
            throws ServletException, IOException {  
  
        String username = req.getParameter("username");  
  
        SqlSession session = factory.openSession();  
        UserMapper mapper = session.getMapper(UserMapper.class);  
  
        // ❌ 用户输入直接进入 SQL        List<User> users = mapper.findUserByName(username);  
  
        resp.setContentType("text/html;charset=utf-8");  
        PrintWriter out = resp.getWriter();  
  
        if (!users.isEmpty()) {  
            out.println("登录成功！");  
            for (User u : users) {  
                out.println("<br>用户名：" + u.getUsername());  
            }  
        } else {  
            out.println("登录失败！");  
        }  
  
        session.close();  
    }  
}
```

***${}不安全 #{}安全***

## Hibernate

### 项目结构

```
hibernate-servlet-demo
├── pom.xml
└── src
    └── main
        ├── java
        │   └── com
        │       └── demo
        │           ├── entity
        │           │   └── User.java
        │           ├── util
        │           │   └── HibernateUtil.java
        │           └── servlet
        │               └── LoginServlet.java
        │
        ├── resources
        │   └── hibernate.cfg.xml
        │
        └── webapp
            └── WEB-INF
                └── web.xml   （可选）

```

### User.java

```
package com.demo.entity;

import javax.persistence.*;

@Entity
@Table(name = "users")
public class User {

    @Id
    @GeneratedValue(strategy = GenerationType.IDENTITY)
    private Integer id;

    private String username;
    private String password;

    // getter / setter
    public Integer getId() {
        return id;
    }

    public void setId(Integer id) {
        this.id = id;
    }

    public String getUsername() {
        return username;
    }

    public void setUsername(String username) {
        this.username = username;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }
}

```

### 工具类

```
package com.demo.util;

import org.hibernate.SessionFactory;
import org.hibernate.cfg.Configuration;

public class HibernateUtil {

    private static final SessionFactory sessionFactory;

    static {
        try {
            sessionFactory = new Configuration()
                    .configure()   // 默认加载 hibernate.cfg.xml
                    .buildSessionFactory();
        } catch (Throwable ex) {
            throw new ExceptionInInitializerError(ex);
        }
    }

    public static SessionFactory getSessionFactory() {
        return sessionFactory;
    }
}

```

### 配置文件

```
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE hibernate-configuration PUBLIC
        "-//Hibernate/Hibernate Configuration DTD 3.0//EN"
        "http://hibernate.sourceforge.net/hibernate-configuration-3.0.dtd">

<hibernate-configuration>
    <session-factory>

        <!-- 数据库连接 -->
        <property name="hibernate.connection.driver_class">
            com.mysql.cj.jdbc.Driver
        </property>

        <property name="hibernate.connection.url">
            jdbc:mysql://localhost:3306/hibernate_test?useSSL=false&amp;serverTimezone=UTC
        </property>

        <property name="hibernate.connection.username">root</property>
        <property name="hibernate.connection.password">root</property>

        <!-- Hibernate 行为 -->
        <property name="hibernate.dialect">
            org.hibernate.dialect.MySQL8Dialect
        </property>

        <property name="hibernate.show_sql">true</property>
        <property name="hibernate.format_sql">true</property>

        <!-- 映射实体 -->
        <mapping class="com.demo.entity.User"/>

    </session-factory>
</hibernate-configuration>

```

### Servlet

```
package com.demo.servlet;

import com.demo.entity.User;
import com.demo.util.HibernateUtil;
import org.hibernate.Session;
import org.hibernate.query.Query;

import javax.servlet.ServletException;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.*;
import java.io.IOException;
import java.io.PrintWriter;
import java.util.List;

@WebServlet("/login")
public class LoginServlet extends HttpServlet {

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp)
            throws ServletException, IOException {

        String username = req.getParameter("username");

        Session session = HibernateUtil.getSessionFactory().openSession();

        // ✅ 正常写法（安全）
        String hql = "from User where username = :username";
        Query<User> query = session.createQuery(hql, User.class);
        query.setParameter("username", username);

        List<User> users = query.list();
        session.close();

        resp.setContentType("text/html;charset=utf-8");
        PrintWriter out = resp.getWriter();

        if (!users.isEmpty()) {
            out.println("登录成功<br>");
            for (User u : users) {
                out.println("用户名：" + u.getUsername() + "<br>");
            }
        } else {
            out.println("登录失败");
        }
    }
}

```

***预编译：执行架构逻辑不改***

