## Vue-Xss

```
<div v-html="displayContent"></div>
```

使用{{}}来渲染不会产生xss


## React-Xss

```
<div dangerouslySetInnerHTML={{__html: displayedInput}}/>
```
直接使用{displayedInput}来显示不会产生XSS

## Electron-XSS

```
const displayArea = document.getElementById('displayArea');
displayArea.innerHTML = input;
```

使用 textContent 代替 innerHTML 来显示文本不会产生XSS

## MXss

参考 https://mp.weixin.qq.com/s/31zaBzZ1e6rNobYCrn7Qhg

模拟

```
https://portswigger-labs.net/mxss/

<math><mtext><table><mglyph><style><!--</style><img title="--&gt;&lt;img src=1 onerror=alert(1)&gt;">
```

因为不同应用的解析差异产生的XSS

https://www.fooying.com/the-art-of-xss-1-introduction/

## UXss

UXSS利用浏览器或者浏览器扩展漏洞来制造产生XSS并执行代码的攻击类型。

复盘：

MICROSOFT EDGE uXSS CVE-2021-34506

Edge浏览器翻译功能导致JS语句被调用执行

https://www.bilibili.com/video/BV1fX4y1c7rX

https://mp.weixin.qq.com/s/rR2feGeuRt3hOFPkV3_6Ow