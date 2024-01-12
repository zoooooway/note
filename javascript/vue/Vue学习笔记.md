# Vue 学习笔记
> 参阅: 
> vue 官方教程: https://v2.cn.vuejs.org/v2/guide/

## 基础知识

### 模板

### 组件

#### slot
##### 具名插槽
请看以下示例:
```html
<!-- <base-layout> 组件 -->
<div class="container">
  <header>
      <!-- 我们希望把页头放这里 -->
      <!-- 提供的插槽 -->
      <slot name="header"></slot>
  </header>
  <main>
      <!-- 我们希望把主要内容放这里 -->
      <!-- 一个不带 name 的 <slot> 出口会带有隐含的名字“default”。 -->
      <slot></slot>
  </main>
  <footer>
      <!-- 我们希望把页脚放这里 -->
      <slot name="footer"></slot>
  </footer>
</div>
```

```html
<!-- 在使用 <base-layout> 组件的地方指定插槽的内容 -->
<base-layout>
  <template v-slot:header>
    <h1>Here might be a page title</h1>
  </template>

  <p>A paragraph for the main content.</p>
  <p>And another one.</p>

  <template v-slot:footer>
    <p>Here's some contact info</p>
  </template>
</base-layout>
```

现在 <template> 元素中的所有内容都将会被传入相应的插槽。任何没有被包裹在带有 v-slot 的 <template> 中的内容都会被视为**默认插槽**的内容。
> 注意，如果具名插槽没有匹配的插槽内容传入，则该插槽不会渲染

## Vue Router
Vue Router 提供路由功能，使得应用程序在多个组件直接跳转更加优雅。

### 导航守卫

#### 完整的导航解析流程
1. 导航被触发。
2. 在失活的组件里调用 beforeRouteLeave 守卫。
3. 调用全局的 beforeEach 守卫。
4. 在重用的组件里调用 beforeRouteUpdate 守卫 (2.2+)。
5. 在路由配置里调用 beforeEnter。
6. 解析异步路由组件。
7. 在被激活的组件里调用 beforeRouteEnter。
8. 调用全局的 beforeResolve 守卫 (2.5+)。
9. 导航被确认。
10. 调用全局的 afterEach 钩子。
11. 触发 DOM 更新。
12. 调用 beforeRouteEnter 守卫中传给 next 的回调函数，创建好的组件实例会作为回调函数的参数传入。

## Vuex

## Vue CLi

## AntDesign
1. 使用命令行进行初始化
   ```shell
   vue create antd-demo
   ```
2. 执行 `npm run serve` 查看项目运行情况
3. 使用组件 
   ```shell
   npm i --save ant-design-vue
   ```
4. 使用 `babel-plugin-import` 来按需加载组件
    ```json
    // .babelrc or babel-loader option
    {
    "plugins": [
        ["import", { "libraryName": "ant-design-vue", "libraryDirectory": "es", "style": "css" }] // `style: true` 会加载 less 文件
    ]
    }    
    ```
    然后只需从 ant-design-vue 引入模块即可，无需单独引入样式。等同于下面手动引入的方式。
    ```js
    // babel-plugin-import 会帮助你加载 JS 和 CSS
    import { DatePicker } from 'ant-design-vue';
     ```
5. 