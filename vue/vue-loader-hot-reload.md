---
description: >-
  官网中介绍： webpack-dev-server --hot 可以在 CLI 中启动热更新模式。但是高阶用户可能希望移步 vue-loader 内部使用的
  vue-hot-reload-api 继续查阅。究竟是 vue-hot-reload-api 里面有什么，让我们一起一探究竟。
---

# Vue-Loader Hot Reload

### Hot Reload 触发时机

```javascript
const needsHotReload = (
    !isServer &&
    !isProduction &&
    (descriptor.script || descriptor.template) &&
    options.hotReload !== false
)
```

* **isServer**  判断当前是浏览器环境还是Node环境；
* **isProduction** 判断只有是开发环境才进行HMR；
* **descriptor** 对象从 **@vue/component-compiler-utils** 提供的 **parse** 函数处理所得。
* **options** 从 **loader-utils** 返回值中所得。

### Loader-utils

![](../.gitbook/assets/image%20%283%29.png)

**loader-utils** 本身是**webpack**提供的插理解是**webpack**本身提供的一些工具方法。

![](../.gitbook/assets/image%20%285%29.png)

不难看出，**loaderUtils.getOptions** 方法本质是序列化 **webpack** 运行时的上下文。

### vue-hot-reload-api 

通过 Debug 可以宏观意义上的去体会 **vue-hot-reload-api** 提供的能力。

![](../.gitbook/assets/image%20%284%29.png)

#### 钩子 API 汇总

| 名称 | 类型 | 作用 |
| :--- | :--- | :--- |
| compatible | Boolen | vue-hot-reload-api 只作用于 vue 2.x 版本 |
| createRecord | Function | 以模块ID为key， |
| install | Function | 将 vue 注册到 vue-hot-reload-api  |
| isRecorded | Function |  |
| reload | Function |  |
| rerender | Function |  |

```javascript
const map = Object.create(null)
if (typeof window !== 'undefined') {
  window.__VUE_HOT_MAP__ = map
}
```

源码中，hot-reload 会缓存在一个对象中。并且对外暴露了更新组件时对象内部的结构。如下如所示：

![](../.gitbook/assets/image.png)



