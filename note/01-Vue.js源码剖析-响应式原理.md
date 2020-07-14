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
