![语言模板支持](./images/模板.jpg)

## SSTI利用思路

1. 确定模板引擎

- 参数看报错
- 看模板语法
- 引擎寻找构造

2. 构造payload的思路

- 寻找可用对象（比如字符串、字典，或者已给出的对象）
- 通过可用对象寻找原生对象（object）
- 利用原生对象实例化目标对象（比如os）

3. 执行代码

如果手工有困难，可用使用Tqlmap或SSTImap代替

## 项目

https://github.com/epinna/tplmap

https://github.com/vladko312/SSTImap

## SRC报告

https://forum.butian.net/share/1229
