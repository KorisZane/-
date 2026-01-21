## Vue下载

```
npm install vite
npm create vue@latest
```

## 启动项目

```
安装依赖：npm install

开发启动：npm run dev

打包构建：npm run build
```

## 安全

```
<template>

<div>

<h1>XSS 漏洞演示</h1>

<input v-model="userInput" placeholder="输入你的内容" />

<button @click="showContent">显示内容</button>

<div v-html="displayContent"></div>

</div>

</template>

<script>

export default {

data() {

return {

userInput: '', // 用户输入

displayContent: '' // 显示的内容

};

},

methods: {

showContent() {

// 直接将用户输入的内容渲染到页面

this.displayContent = this.userInput;

}

}

};

</script>

<style>

#app {

font-family: Avenir, Helvetica, Arial, sans-serif;

text-align: center;

margin-top: 60px;

}

</style>****
```

用v-html做显示容易xss 使用文本插值（{{}}）代替 v-html没有xss危险

#### WebPack+VueJS源码泄露

// vue.config.js

export default defineConfig({

plugins: [vue()],

build: {

sourcemap: true, // 如果需要生成 Source Map

},

})

npm run build