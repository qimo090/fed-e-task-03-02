# 课程目标

- Vue.js 的静态成员和实例成员初始化过程
- 首次渲染的过程
- **数据响应式原理**

# 准备工作

## Vue 源码的获取

- 项目地址：[https://github.com/vuejs/vue](https://github.com/vuejs/vue)
- Fork 一份代码到自己仓库，再克隆到本地，可以自己写注释提交到 GitHub
- 为什么分析 Vue 2.6

  - 到目前为止 Vue3.0 的正式版还没有发布
  - 新版本发布后，现有项目不会升级到 3.0，2.x 还有很长的一段过渡期
  - 3.0 项目地址：[https://github.com/vuejs/vue-next](https://github.com/vuejs/vue-next)

## 源码目录结构

```
src
  |-comiler   编译相关
  |-core      Vue 核心库
  |-platforms 平台相关代码
  |-server    SSR，服务端渲染
  |-sfc       .vue 文件编译为 js 对象
  |_shared    公共的代码
```

## 了解 Flow

- 官网 [https://flow.org/](https://flow.org/)
- JavaScript 的 **静态类型检查器**
- Flow 的静态类型检查错误是通过静态类型推断实现的

  - 文件开头通过 `// @flow` 或者 `/* @flow */` 声明

  ```flow
  /* @flow */
  function square(n: number): number {
    return n * n
  }
  square('2') // Error!
  ```

# 调试设置

## 打包

- 打包工具 Rollup
  - Vue.js 源码的打包工具使用的是 Rollup，比 Webpack 轻量
  - Webpack 把所有文件当作模块，Rollup 只处理 js 文件，更适合在 Vue.js 这样的库中使用
  - Rollup 打包不会生成冗余的代码
- 安装依赖

  ```shell
  npm i
  ```

- 设置 sourcemap

  - package.json 文件中 dev 脚本中添加参数 --sourcemap

    ```json
    {
      "scripts": {
        "dev": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:web-full-dev"
      }
    }
    ```

- 执行 dev
  - `npm run dev` 执行打包，用的是 Rollup，-w 参数是监听文件的变化，文件变化自动重新打包
  - 结果

# Vue 的不同构建版本

- `npm run build` 重新打包所有文件
- [官方文档 - 对不同构建版本的解释](https://cn.vuejs.org/v2/guide/installation.html#%E5%AF%B9%E4%B8%8D%E5%90%8C%E6%9E%84%E5%BB%BA%E7%89%88%E6%9C%AC%E7%9A%84%E8%A7%A3%E9%87%8A)
- `dist/README.md`

在 NPM 包的 dist/ 目录你将会找到很多不同的 Vue.js 构建版本。这里列出了它们之间的差别：
|#|UMD|CommonJS|ES Module|
|--|--|--|--|
|Full|vue.js|vue.common.js|vue.esm.js|
|Runtime-only|vue.runtime.js|vue.runtime.common.js|vue.runtime.esm.js|
|Full(production)|vue.min.js|||
|Runtime-only(production)|vue.runtime.min.js|||

## 术语

- **完整版**：同时包含 **编译器** 和 **运行时** 的版本。

- **编译器**：用来将模板字符串编译成为 JavaScript 渲染函数的代码。

- **运行时**：用来创建 Vue 实例、渲染并处理虚拟 DOM 等的代码。基本上就是除去编译器的其它一切。

- **[UMD](https://github.com/umdjs/umd)**：UMD 版本可以通过 `<script>` 标签直接用在浏览器中。`vue.js` 默认文件就是运行时 + 编译器的 UMD 版本。

* **[CommonJS](http://wiki.commonjs.org/wiki/Modules/1.1)**：CommonJS 版本用来配合老的打包工具比如 [Browserify](http://browserify.org/) 或 [webpack 1](https://webpack.github.io/)。这些打包工具的默认文件 (`pkg.main`) 是只包含运行时的 CommonJS 版本 (`vue.runtime.common.js`)。

- [ES Module](https://exploringjs.com/es6/ch_modules.html)：从 2.6 开始 Vue 会提供两个 ES Modules (ESM) 构建文件：

  - 为打包工具提供的 ESM：为诸如 [webpack 2](https://webpack.js.org/) 或 [Rollup](https://rollupjs.org/) 提供的现代打包工具。ESM 格式被设计为可以被静态分析，所以打包工具可以利用这一点来进行“tree-shaking”并将用不到的代码排除出最终的包。为这些打包工具提供的默认文件 (`pkg.module`) 是只有运行时的 ES Module 构建 (`vue.runtime.esm.js`)。

  - 为浏览器提供的 ESM (2.6+)：用于在现代浏览器中通过 `<script type="module">` 直接导入。

# 寻找入口文件

查看 `dist/vue.js` 的构建过程

## 执行构建

```shell
npm run dev
# "dev": "rollup -w -c scripts/config.js --environment TARGET:web-full-dev"
# --environment TARGET:web-full-dev 设置环境变量 TARGET
```

- `scripts/config.js` 的执行过程
  - 作用：生成 rollup 构建的配置文件
  - 使用环境变量 TARGET=web-full-dev

```javascript
// 判断环境变量是否有 TARGET
// 如果有的话，使用 genConfig() 生成 rollup 配置文件
if (process.env.TARGET) {
  module.exports = genConfig(process.env.TARGET)
} else {
  // 否则获取全部配置
  exports.getBuild = genConfig
  exports.getAllBuilds = () => Object.keys(builds).map(genConfig)
}
```

# 从入口开始

`src/platforms/web/entry-runtime-with-compiler.js`

## 通过查看源码解决下面问题

观察以下代码，通过阅读源码，回答在页面上输出的结果

```javascript
const vm = new Vue({
  el: '#app',
  template: '<h3>Hello template</h3>',
  render(h) {
    return h('h4', 'Hello render')
  },
})
```

- 阅读源码记录
  - `el` 不能是 `body` 或 `html` 标签
  - 如果没有 `render`，就会把 `template` 转化成 `render` 函数，`render` 优先级高
  - 如果有 `render()`，直接调用 `mount` 挂载 DOM

```javascript
// --- src/platforms/web/entry-runtime-with-compiler.js
// 1. el 不能是 body 或者 html
if (el === document.body || el === document.documentElement) {
  process.env.NODE_ENV !== 'production' &&
    warn(
      `Do not mount Vue to <html> or <body> - mount to normal elements instead.`
    )
  return this
}
const options = this.$options
// resolve template/el and convert to render function
if (!options.render) {
  // 2. 把 template/el 转换成 render 函数
  // ...
}
// 3. 调用 mount 方法，挂载 DOM
return mount.call(this, el, hydrating)
```

- 调试代码
  调试的方法

```javascript
const vm = new Vue({
  el: '#app',
  template: '<h3>Hello template</h3>',
  render(h) {
    return h('h4', 'Hello render')
  },
})
```

![代码调试](https://tva1.sinaimg.cn/large/007S8ZIlly1ggt55e3i6jj31m20m612p.jpg)

## 问题

- Vue 的构造函数在哪？
- Vue 实例的成员 / Vue 的静态成员哪里来的？

# Vue 初始化的过程

## 四个导出 Vue 的模块

- src/**platforms/web**/entry-runtime-with-compiler.js
  - web 平台相关的代码入口
  - 重写了平台相关的 `$mount()` 方法
  - 注册了 `Vue.compile()` 方法，传递一个 HTML 字符串返回 `render` 函数
- src/**platforms/web**/runtime/index.js
  - web 平台相关
  - 注册和平台相关的全局指令：`v-model`, `v-show`
  - 注册和平台相关的全局组件：`transition`, `transition-group`
  - 全局方法
    - `__patch__` - 把虚拟DOM 转换成真实 DOM
    - `$mount` - 挂载方法
- src/**core**/index.js
  - 与平台无关
  - 设置了 Vue 的静态方法，`initGlobalAPI(Vue)`
- src/**core**/instance/index.js
  - 与平台无关
  - 定义了构造函数，调用了 `this._init(options)` 方法
  - 给 Vue 中混入了常用的实例成员
