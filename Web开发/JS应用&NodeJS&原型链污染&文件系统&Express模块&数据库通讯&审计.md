## nodejs安装

文档参考 https://www.runoob.com/nodejs/nodejs-tutorial.html

官网 https://nodejs.org/en

## 代码演示

```
var fs = require("fs");  
fs.readFile('project.json', 'utf8', function(err, data) {  
    if(err) throw err;  
    console.log(data);  
})
```

## Express框架

Express是一个简洁而灵活的node.js Web应用框架

npm install express

```
const fs = require('fs');  
const express = require('express')  
const app = express()  
app.get('/', function (req, res) {  
    var name = req.query.x;  
    res.send(name)  
    fs.readFile('1.txt', 'utf8', function (err, data) {  
        if (err) return res.status(500).send(err)  
        else console.log(data);  
    })  
})  
var server = app.listen(9003,function(){  
    var host=server.address().address;  
    var port=server.address().port;  
    console.log("应用实例，访问地址为 http://%s:%s", host, port );  
})
```

## Mysql

npm install mysql

```
const mysql = require('mysql');  
var connection = mysql.createConnection({  
    host: 'localhost',  
    user: 'root',  
    password: 'root',  
    database: 'test',  
})  
connection.connect()  
var sql = "select * from admin where id='1'";  
connection.query(sql, function (err, result) {  
    if (err) throw err;  
    else console.log("result", result);  
})
```

## RCE

```
const child_process = require('child_process');  
  
child_process.exec("calc")  
child_process.spawnSync("mstsc");  
  
eval('child_process.spawnSync("cap")')
```

## CTF方向

参考 https://f1veseven.github.io/2022/04/03/ctf-nodejs-zhi-yi-xie-xiao-zhi-shi/

```
// foo是一个简单的JavaScript对象  
let foo = {bar: 1}  
  
// foo.bar 此时为1  
console.log(foo.bar)  
  
// 修改foo的原型（即Object）  
foo.__proto__.bar = 2  
  
// 由于查找顺序的原因，foo.bar仍然是1  
console.log(foo.bar)  
  
// 此时再用Object创建一个空的zoo对象  
let zoo = {}  
  
// 查看zoo.bar，此时bar为2  
console.log(zoo.bar)
```

## 代码审计案例

审计参考:https://mp.weixin.qq.com/s/mKOlTQclji-oEB5x_bMEMg

部署：https://hellosean1025.github.io/yapi/devops/index.html

手工bug：

https://github.com/YMFE/yapi/issues/2180

Docker：https://github.com/MyHerux/code-note/blob/master/Program/%E5%B7%A5%E5%85%B7%E7%AF%87/Yapi/%E4%BD%BF%E7%94%A8DockerCompose%E6%9E%84%E5%BB%BA%E9%83%A8%E7%BD%B2Yapi.md

3、代码审计：https://mp.weixin.qq.com/s/fe_Kp7fOUiMXLWTY1KxSqg