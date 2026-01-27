## AST

简单来说AST就是源代码的抽象语法结构的树状表示，树上的每个节点都表示源代码的一种结构，这种数据结构其实可以类比成一个大的JSON对象。

官网 https://astexplorer.net/

参考 

https://mp.weixin.qq.com/s/bOc8PYbFdTyFRQcfSppo8w

https://mp.weixin.qq.com/s/rURCR085HiojW2_67enJkA

## 第一词法分析

一段代码首先会被分解成段段有意义的词法单元，

比如const name="qc”这段代码，它可被折分成四部分：

1、const

2、name

3、=

4、qc

每个部分都具备一定的含义。

## 第二语法分析

接着编译器会尝试对一个个法单元进行语法分析，将其转换为能代表程序语法结构的数据结构。

比如

1、const就被分析为VariableDeclaration类型，代表变量声明的具体定义；

2、name就被分析为ldentifier类型，代表一个标识符

3、qc就被分析为Literal类型，代表文本内容；

第三指令生成

最后将AST转为可执行指令并执行

Literal:简单理解就是字面量，比如3、"abc"、null这些都是基本的字面量。在代码中又细分为数字字面量，字符字面量等；

Declarations:声明，通常声明方法或者变量。

Expressions:表达式，通常有两个作用：一个是放在赋值语句的右边进行赋值，另外还可以作为方法的参数。

Statemonts:语句。

Identifier:标识符，指代变量名，比如上述例子中的name就是ldentifier。

Classes:类，代表一个类的定义。

Functions:方法声明。

Modules:模块，可以理解为一个Node.js模块。

Program:程序，整个代码可以称为Program。

OB混淆还原：

https://obfuscator.io/

https://deobfuscate.io/

https://webcrack.netlify.app/

https://deli-c1ous.github.io/javascript-deobfuscator/