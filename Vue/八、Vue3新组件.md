# 八、Vue3 新组件

## 8.1. 【Teleport】

* 什么是Teleport？—— Teleport 是一种能够将我们的**组件html结构**移动到指定位置的技术。

```html
<teleport to='body' >
    <div class="modal" v-show="isShow">
      <h2>我是一个弹窗</h2>
      <p>我是弹窗中的一些内容</p>
      <button @click="isShow = false">关闭弹窗</button>
    </div>
</teleport>

```

## 8.2. 【Suspense】

* 等待异步组件时渲染一些额外内容，让应用有更好的用户体验
* 使用步骤：
  * 异步引入组件
  * 使用`Suspense`包裹组件，并配置好`default` 与 `fallback`

```tsx
import { defineAsyncComponent,Suspense } from "vue";
const Child = defineAsyncComponent(()=>import('./Child.vue'))
```

`<Child/>`是一个包含了异步请求的子组件

```vue
<template>
  <div class="child">
    <h2>我是Child组件</h2>
    <h3>当前求和为：{{ sum }}</h3>
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue'
import axios from 'axios'

let sum = ref(0)
let { data: { content } } = await axios.get('https://api.uomg.com/api/rand.qinghua?format=json')
console.log(content)

</script>

```

```vue
<template>
    <div class="app">
        <h3>我是App组件</h3>
        <Suspense>
          <template v-slot:default>
            <Child/>
          </template>
          <template v-slot:fallback>
            <h3>加载中.......</h3>
          </template>
        </Suspense>
    </div>
</template>

```

## 8.3.【全局API转移到应用对象】

* `app.component`	注册全局组件
* `app.config`     配置对象
* `app.directive`     注册全局指令
* `app.mount`
* `app.unmount`
* `app.use`

## 8.4.【其他】

* 过渡类名 `v-enter` 修改为 `v-enter-from`、过渡类名 `v-leave` 修改为 `v-leave-from`。
* `keyCode` 作为 `v-on` 修饰符的支持。
* `v-model` 指令在组件上的使用已经被重新设计，替换掉了 `v-bind.sync。`
* `v-if` 和 `v-for` 在同一个元素身上使用时的优先级发生了变化。
* 移除了`$on`、`$off` 和 `$once` 实例方法。
* 移除了过滤器 `filter`。
* 移除了`$children` 实例 `propert`。......


> 更新: 2026-03-31 18:21:19  
> 原文: <https://www.yuque.com/thinkspace/bamia2/om2kfrcbw44sxq62>