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
  ├─compiler  编译相关
  ├─core Vue  核心库
  ├─platforms 平台相关代码
  ├─server    SSR，服务端渲染
  ├─sfc       .vue 文件编译为 js 对象
  └─shared    公共的代码
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

  - package.json 文件中 dev 脚本中添加参数 `--sourcemap`
    ```json
    {
      "scripts": {
        "dev": "rollup -w -c scripts/config.js --sourcemap --environment TARGET:web-full-dev"
      }
    }
    ```

- 执行 dev
  - `npm run dev` 执行打包，用的是 Rollup，`-w` 参数是监听文件的变化，文件变化自动重新打包
  - 结果
    ![dist](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh7syhigrbj30k60ouq5t.jpg)
  - 调试
    - examples 的示例中引入的 vue.min.js 改为 vue.js
    - 打开 Chrome 的调试工具中的 source
      ![source](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh7t3wt2taj30og0fydhc.jpg)

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

# Runtime + Compiler vs. Runtime-only

```javascript
// Compiler
// 需要编译器，把 template 转换成 render 函数
// const vm = new Vue({
//   el: '#app',
//   template: '<h1>{{ msg }}</h1>',
//   data: {
//     msg: 'Hello Vue',
//   }
// })

// Runtime
// 不需要编译器
const vm = new Vue({
  el: '#app',
  render(h) {
    return h('h1', this.msg)
  },
  data: {
    msg: 'Hello Vue',
  },
})
```

- 推荐使用运行时版本，因为运行时版本相比完整版体积要小约 30%
- 基于 Vue-Cli 创建的项目默认使用的是 `vue.runtime.esm.js`
  - 通过查看 webpack 的配置文件
    ```shell
    vue inspect > output.js
    ```
- **注意**：`*.vue` 文件中的模板是在构建时预编译的，最终打包后的结果不需要编译器，只需要运行时版本即可

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
  - 使用环境变量 `TARGET=web-full-dev`

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

- `genConfig(name)`
  - 根据环境变量 `TARGET` 获取配置信息
  - `builds[name]` 获取生成配置的信息
  ```js
  // Runtime+compiler development build (Browser)
  'web-full-dev': {
    entry: resolve('web/entry-runtime-with-compiler.js'),
    dest: resolve('dist/vue.js'),
    format: 'umd',
    env: 'development',
    alias: { he: './entity-decoder' },
    banner
  },
  ```
- `resolve()`
  - 获取入口和出口文件的绝对路径
  ```js
  const aliases = require('./alias')
  const resolve = p => {
    const base = p.split('/')[0]
    if (aliases[base]) {
      return path.resolve(aliases[base], p.slice(base.length + 1))
    } else {
      return path.resolve(__dirname, '../', p)
    }
  }
  ```

## 结果

- 把 `src/platforms/web/entry-runtime-with-compiler.js` 构建成 `dist/vue.js`，如果设置 `--sourcemap` 会生成 `vue.js.map`
- `src/platforms` 文件夹下是 Vue 可以构建成不同平台下使用的库，目前有 `weex` 和 `web`，还有服务器端渲染的库

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

# Vue 的构造函数在哪

- `src/platforms/web/entry-runtime-with-compiler.js` 中引用了 `./runtime/index`
- `src/platforms/web/runtime/index.js`

  - 设置 Vue.config
  - 设置平台相关的指令和组件
    - 指令：`v-model`, `v-show`
    - 组件：`transition`, `transition-group`
  - 设置平台相关的 `__patch__` 方法（打补丁方法，对比新旧的 VNode）
  - **设置 \$mount 方法，挂载 DOM**

    ```javascript
    // install platform runtime directives & components
    extend(Vue.options.directives, platformDirectives)
    extend(Vue.options.components, platformComponents)

    // install platform patch function
    Vue.prototype.__patch__ = inBrowser ? patch : noop

    // public mount method
    Vue.prototype.$mount = function (
      el?: string | Element,
      hydrating?: boolean
    ): Component {
      el = el && inBrowser ? query(el) : undefined
      return mountComponent(this, el, hydrating)
    }
    ```

  - `src/platforms/web/runtime.index.js` 中引用了 `core/index`
  - `src/core/index.js`
    - 定义了 Vue 的静态方法
    - `initGlobalAPI(Vue)`
  - `src/core/index.js` 中引用了 `./instance/index`
  - `src/core/instance/index.js`

    - 定义了 Vue 的构建函数

    ```js
    function Vue(options) {
      if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
        warn('Vue is a constructor and should be called with the `new` keyword')
      }
      // 调用 _init() 方法
      this._init(options)
    }

    // 注册 vm 的 _init() 方法，初始化 vm
    initMixin(Vue)
    // 注册 vm 的 $data/$props/$set/$delete/$watch
    stateMixin(Vue)
    // 初始化事件相关方法
    // $on/$once/$off/$emit
    eventsMixin(Vue)
    // 初始化生命周期相关的混入方法
    // _update/$forceUpdate/$destroy
    lifecycleMixin(Vue)
    // 混入 render
    // $nextTick/_render
    renderMixin(Vue)
    ```

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
    - `__patch__` - 把虚拟 DOM 转换成真实 DOM
    - `$mount` - 挂载方法
- src/**core**/index.js
  - 与平台无关
  - 设置了 Vue 的静态方法，`initGlobalAPI(Vue)`
- src/**core**/instance/index.js
  - 与平台无关
  - 定义了构造函数，调用了 `this._init(options)` 方法
  - 给 Vue 中混入了常用的实例成员

# Vue 的初始化

## `src/core/global-api/index.js`

- 初始化 Vue 的静态方法

  ```ts
  // --- src/core/index.js
  // 注册 Vue 的静态属性/方法
  initGlobalAPI(Vue)
  // --- src/core/global-api/index.js
  // 初始化 Vue.config 对象
  Object.defineProperty(Vue, 'config', configDef)

  // exposed util methods.
  // NOTE: these are not considered part of the public API - avoid relying on
  // them unless you are aware of the risk.
  // 这些工具方法不视作全局 API 的一部分，除非你已经意识到某些风险，否则不要去依赖它们
  Vue.util = {
    warn,
    extend,
    mergeOptions,
    defineReactive,
  }
  // 静态方法 set/delete/nextTick
  Vue.set = set
  Vue.delete = del
  Vue.nextTick = nextTick

  // 2.6 explicit observable API
  // 让一个对象响应化
  Vue.observable = <T>(obj: T): T => {
    observe(obj)
    return obj
  }
  // 初始化 Vue.options 对象，并给其扩展
  // components/directives/filters
  Vue.options = Object.create(null)
  ASSET_TYPES.forEach(type => {
    Vue.options[type + 's'] = Object.create(null)
  })

  // this is used to identify the "base" constructor to extend all plain-object
  // components with in Weex's multi-instance scenarios.
  Vue.options._base = Vue

  // 设置 keep-alive 组件
  extend(Vue.options.components, builtInComponents)

  // 注册 Vue.use() 用来注册插件
  initUse(Vue)
  // 注册 Vue.mixin() 实现混入
  initMixin(Vue)
  // 注册 Vue.extend() 基于传入的 options 放回一个组件的构造函数
  initExtend(Vue)
  // 注册 Vue.directive() Vue.component() Vue.filter()
  initAssetRegisters(Vue)
  ```

- `src/core/instance/index.js`

  - 定义 Vue 的构造函数
  - 初始化 Vue 的实例成员

  ```js
  // 此处不用 class 的原因是为了后续方便给 Vue 实例混入实例成员
  function Vue(options) {
    if (process.env.NODE_ENV !== 'production' && !(this instanceof Vue)) {
      warn('Vue is a constructor and should be called with the `new` keyword')
    }
    // 调用 _init() 方法
    this._init(options)
  }

  // 注册 vm 的 _init() 方法，初始化 vm
  initMixin(Vue)
  // 注册 vm 的 $data/$props/$set/$delete/$watch
  stateMixin(Vue)
  // 初始化事件相关方法
  // $on/$once/$off/$emit
  eventsMixin(Vue)
  // 初始化生命周期相关的混入方法
  // _update/$forceUpdate/$destroy
  lifecycleMixin(Vue)
  // 混入 render
  // $nextTick/_render
  renderMixin(Vue)
  ```

- `initMixin(Vue)`

  - 初始化 `_init()` 方法

  ```ts
  // --- src/core/instance/init.js
  export function initMixin(Vue: Class<Component>) {
    // 给 Vue 实例增加 _init() 方法
    // 合并 options，初始化操作
    Vue.prototype._init = function (options?: Object) {
      const vm: Component = this
      // a uid
      vm._uid = uid++
      // a flag to avoid this being observed
      // 如果是 Vue 实例就不需要 observe
      vm._isVue = true
      // merge options
      // 合并 options
      if (options && options._isComponent) {
        // optimize internal component instantiation
        // since dynamic options merging is pretty slow, and none of the
        // internal component options needs special treatment.
        initInternalComponent(vm, options)
      } else {
        vm.$options = mergeOptions(
          resolveConstructorOptions(vm.constructor),
          options || {},
          vm
        )
      }
      /* istanbul ignore else */
      if (process.env.NODE_ENV !== 'production') {
        initProxy(vm)
      } else {
        vm._renderProxy = vm
      }
      // expose real self
      vm._self = vm
      // vm 的生命周期相关变量初始化
      // $children/$parent/$root/$refs
      initLifecycle(vm)
      // vm 的事件监听初始化，福组建绑定在当前组件上的事件
      initEvents(vm)
      // vm 的编译 render 初始化
      // $slots/$scopedSlots/_c/$createElement/$attrs/$listeners
      initRender(vm)
      // beforeCreate 生命钩子的回调
      callHook(vm, 'beforeCreate')
      // 把 inject 的成员注入到 vm 上
      initInjections(vm) // resolve injections before data/props
      // 初始化状态 vm 的 _props/methods/_data/computed/watch
      initState(vm)
      // 初始化 provide
      initProvide(vm) // resolve provide after data/props
      // created 生命钩子的回调
      callHook(vm, 'created')

      /* istanbul ignore if */
      if (process.env.NODE_ENV !== 'production' && config.performance && mark) {
        vm._name = formatComponentName(vm, false)
        mark(endTag)
        measure(`vue ${vm._name} init`, startTag, endTag)
      }
      // 如果没有提供 el，调用 $mount() 挂载
      if (vm.$options.el) {
        vm.$mount(vm.$options.el)
      }
    }
  }
  ```

## 首次渲染过程

- Vue 初始化完毕，开始真正的执行
- 调用 new Vue() 之前，已经初始化完毕
- 通过调试代码，记录首次渲染过程

![new Vue](https://tva1.sinaimg.cn/large/007S8ZIlgy1gh7udwumknj30u00usn9j.jpg)

# 数据响应式原理

## 通过查看源码解决下面问题

- `vm.msg = { count: 0 }` 重新给属性赋值，是否是响应式的？
- `vm.arr[0] = 4` 给数组元素赋值，视图是否会更新
- `vm.arr.length = 0` 修改数组的 `length`，视图是否会更新
- `vm.arr.push(4)` 视图是否会更新

## 响应式处理的入口

整个响应式处理的过程是比较复杂的，下面我们先从

- `src/core/instance/init.js`

  - `initState(vm)` vm 状态的初始化
  - 初始化 `_data`, `_props`, `methods` 等

- `src/core/instance/state.js`

  ```javascript
  // 数据的初始化
  if (opts.data) {
    initData(vm)
  } else {
    observe((vm.data = {}), true /* asRootData */)
  }
  ```

# Watcher

- Wathcer 分为三种，Computed Watcher、用户 Watcher（监听器）、**渲染 Watcher**
- 渲染 Watcher 的创建时机
  - `src/core/instance/lifecycle.js`
