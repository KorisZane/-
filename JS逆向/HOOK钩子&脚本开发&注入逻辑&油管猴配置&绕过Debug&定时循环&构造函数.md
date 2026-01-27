## JS HOOK

JS Hook（钩子）是一种通过拦截和修改JavaScript函数或对象行为的技术

主要用于：

*动态分析网页行为

*修改页面功能

*调试和逆向工程

*自动化测试

*安全研究

## hook解释

就是函数覆盖，后面定义的函数会覆盖前面定义的函数

```
<script>  
    function test01() {  
        console.log("test01")  
        test02()  
    }  
    function test02(){  
        console.log("test02")  
        test03()  
    }  
    function test03(){  
        console.log("test03")  
        test04()  
    }  
    function test04(){  
        console.log('test04')  
    }  
    test01()  
</script>

执行结果 test01 test02 test03 test04
```

```
<script>  
    function test01() {  
        console.log("test01")  
        test02()  
    }  
    function test02(){  
        console.log("test02")  
        test03()  
    }  
    function test03(){  
        console.log("test03")  
        test04()  
    }  
    function test04(){  
        console.log('test04')  
    }  
    test01()  

  function test03(){
	  console.log("hello")
  }
</script>

执行结果 test01 test02 hello
```

保存test03函数执行 hook

```
<script>  
    function test01() {  
        console.log("test01")  
        test02()  
    }  
    function test02(){  
        console.log("test02")  
        test03()  
    }  
    function test03(){  
        console.log("test03")  
        test04()  
    }  
    function test04(){  
        console.log('test04')  
    }  
    test01()  
    (function(){  
        const _test03 = test03  
        test03 = function() {  
            console.log("hello")  
            return _test03.apply(this, arguments)  
        }  
    })()  
</script>

执行结果 test01 test02 hello test03 test04
```

(function(){})() 立即执行函数表达式IIFE **创造私有作用域**，避免污染全局变量

return \_test03 确保函数返回值

\_test03.apply(this, arguments) 把当前这个函数收到的 this 和参数，原封不动地转交给原函数执行

***函数hook时机要把握好，只能在test01()调用之前hook，test01()调用后函数调用链已经生成，函数hook不了了***

hook模板

```
(function(){
	const _fn = func
	func = function() {
		你要执行的代码
		return _fn.apply(this, arguments)
	}
})()
```


## HOOK开发前置

```
@name 定义脚本的名称
@namespace 用于区分不同脚本的作用域
@version 定义脚本的版本号
@description 描述脚本的功能或用途
@author 定义脚本的作者
@match 定义脚本的运行匹配规则
@icon 定义脚本的图标
@grant 定义脚本所需的特殊权限
```

@namespace默认即可
@match要写作用的网站：https://www.baidu.com/*

## 案例

### hook固定宽高

```
// ==UserScript==
// @name         Fixed_window_size
// @namespace    https://github.com/0xsdeo/Hook_JS
// @version      2025-01-11
// @description  js获取浏览器高度或宽度时获取到的是脚本中设置的固定值，如果网站js尝试去设置高度和宽度值会直接去设置固定值，并不会按照网站js设置的值去设置。
// @author       0xsdeo
// @match        http://*/*
// @icon         data:image/gif;base64,R0lGODlhAQABAAAAACH5BAEKAAEALAAAAAABAAEAAAICTAEAOw==
// @grant        none
// ==/UserScript==

(function () {
    'use strict';
    let height = 660; // 固定的高度值
    let width = 1366; // 固定的宽度值
    let innerHeight_property_accessor = Object.getOwnPropertyDescriptor(window, "innerHeight"); // 获取innerHeight属性访问器描述符
    let innerHeight_set_accessor = innerHeight_property_accessor.set; // 获取setter
    Object.defineProperty(window, "innerHeight", {
        get: function () {
            // 在这里写你想让hook后的属性在被获取值时执行的代码
            return height;
        },
        set: function () {
            // 在这里写你想让hook后的属性在被设置值时执行的代码
            innerHeight_set_accessor.call(window, height);// 将网站js设置目标属性值时所传入的内容传给原setter设置并返回结果
        }
    });
    let innerWidth_property_accessor = Object.getOwnPropertyDescriptor(window, "innerWidth"); // 获取innerWidth属性访问器描述符
    let innerWidth_set_accessor = innerWidth_property_accessor.set; // 获取setter
    Object.defineProperty(window, "innerWidth", {
        get: function () {
            // 在这里写你想让hook后的属性在被获取值时执行的代码
            return width;
        },
        set: function () {
            // 在这里写你想让hook后的属性在被设置值时执行的代码
            innerWidth_set_accessor.call(window, width);// 将网站js设置目标属性值时所传入的内容传给原setter设置并返回结果
        }
    });
})();
```

### debug byupass

无限debug代码

```
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <title>Title</title>
</head>
<body>
    <script>
        var dbg = new Function("debugger");
        setInterval(dbg, 3000);
    </script>
</body>
</html>
```

```
// ==UserScript==
// @name         定时debug绕过-最终版
// @namespace    http://tampermonkey.net/
// @version      2026-01-22
// @description  强制最快速度注入并拦截
// @author       1x1shy
// @match        *://www.xiaodi8.com/*
// @match        *://127.0.0.1/*
// @match        file:///*
// @run-at       document-start
// @grant        none
// ==/UserScript==

(function() {
    'use strict';

    // 弹窗确认脚本是否生效（测试成功后可删除）
    console.log("油猴脚本已在页面最早期注入");

    const _Function = window.Function;
    window.Function = function(...args) {
        if (args.length > 0 && typeof args[args.length - 1] === 'string') {
            // 增加模糊匹配，防止混淆
            if (args[args.length - 1].toLowerCase().includes("debugger")) {
                console.warn("成功拦截到 debugger 指令！");
                return function() {}; 
            }
        }
        return _Function.apply(this, args);
    };
    
    // 修复原型链，防止代码环境检测
    window.Function.prototype = _Function.prototype;

})();
```

// @run-at       document-start 注入时机 强制要求浏览器：**“在解析 HTML 任何一行代码之前，先执行我的油猴脚本。”**

