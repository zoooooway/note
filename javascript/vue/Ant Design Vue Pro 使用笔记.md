# Ant Design Vue Pro 使用笔记

> 参阅: 
> 项目github地址: https://github.com/vueComponent/ant-design-vue-pro
> 官方文档: https://pro.antdv.com/docs/getting-started
> 项目引用的组件: https://github.com/vueComponent/pro-components

## 简述
记录一下使用 Ant Design Vue Pro 作为项目脚手架的使用流程, 留存以供以后使用时查阅.

## 需要修改的地方

### 项目基本配置

作为新项目的脚手架, 首先应该修改以下几点: 

#### logo
logo 分为网站左侧菜单栏的图标和网站页签的图标, 它们分别在以下位置修改:
* 菜单栏图标: 图标文件是: */src/asserts/logo.svg*
  如果你想更改图片文件的路径, 不使用这个路径, 那么你需要到 */src/layouts/BasicLayout.vue* 里去更改引用图标文件的路径.

* 网站页签图标: 图标文件是: */public/logo.png*
  如果你想更改图片文件的路径, 不使用这个路径, 那么你需要到 */public/index.html* 里去更改引用图标文件的路径.

#### 项目标题
项目标题只需要修改 */src/config/defaultSetting.js* 里 title 的值.

### 业务配置

#### 关闭mock, 与后端服务交互
mock 功能关闭只需要在 */src/main.js* 中注释引入 mock 的代码即可.
替换请求地址为后端服务需要开启 devServer. 找到项目根目录下的 */vue.config.js*, 找到 devServer 配置项, 将其替换为: 
```jsx

```

#### axios request 
脚手架自带的基于 axios 封装的请求工具类属于较为通用的实现, 如果你想要改动 axios 的请求参数, 比如超时时间, 异常处理等, 需要在 */src/utils/request.js* 中改动.

如果你的后端服务使用 `JWT` 进行认证, 你也许会想要更改请求头中的 token header 名称, 具体 header 名称在 */src/store/index.js* 文件中定义. 

#### 响应参数的调整
部分脚手架自带的页面如果想要使用, 可能需要进行调整. 比如登录页面, 登录使用的接口可能会和实际后端服务返回的数据格式不同, 这时需要去具体页面中去调整响应的解析.

这部分工作其实相当多, 脚手架提供的基础功能中, 登录, 路由, 权限这几块功能的参数你都需要根据 `mock` 的数据格式来调整后端接口, 或者调整前端参数的解析. 相对于调整前端, 我更推荐调整后端的参数, 因为后端的参数查找起来很容易, 通常参数都在实体类中, 且使用get/set方法访问, 而前端......, 除非你对这个脚手架十分了解, 那可以考虑.

权限校验可以移除, 但路由相关的功能一般来说仍然会涉及到用户, 角色, 菜单. 所以按 `mock` 方法的参数去实现后端接口无法避免. 简单的关联关系如下:

```
user -> role -> permission(菜单)
```
将 user 与 role 进行关联, role 与 permission 进行关联. permission 中定义所有菜单及其子菜单. 每一项都是一个可访问的路由路径, 并且在一个字段中(比如actions)罗列出其所拥有的所有操作(get, add, del...). 然后在关联关系表 role_permission 中定义角色被分配的权限(permission), 权限应当是上述所有操作(actions)的子集


#### 路由
新增了页面后, 你需要修改路由配置, 才能将新页面挂载到路由上, 从而正常显示和跳转.
路由配置分为静态路由和动态路由, 两种配置的切换在 */src/store/index.js* 中进行修改.


##### 静态路由
默认为静态路由, 即路由完全在前端进行配置, 并不向后端请求路由. 

##### 动态路由
前端只会配置一些基础路由, 比如登录, 404页面, 剩余的路由全部由后端返回. 前端拿到后端返回的该用户的路由后, 将其和基础路由进行合并, 生成该用户的完整路由并将其加载到路由表中.
在配置动态路由时, 你需要修改以下内容:

###### 返回路由表
*/src/router/generator-routers.js* 中定义的路由组件表: `constantRouterComponents`. 该对象中定义了所有路由需要用到的组件, 你可以在此定义新的组件, 并在后端保存路由时, 将 `component` 的值设置为你定义的新的组件.
举例来说, 假如我需要新增一个表单页面, 先在 `constantRouterComponents` 中定义一个组件: `BasicForm: () => import('@/views/form/basicForm')`, 然后后端返回的路由信息中的 `component` 的值就指定为 `BasicForm`.
你也可以选择直接在后端返回的路由信息中指定组件路径, 比如: `component` 的值就指定为 `form/basicForm`, 但不推荐这样做. 路由加载的具体逻辑在 */src/router/generator-routers.js* `generator` 函数中.



#### 引入组件
有时你需要引入一些新的第三方组件, 你需要去修改相关组件加载的配置: 
* 组件按需加载 `/src/main.js` 
* L14 相关代码 `import './core/lazy_use'` / `import './core/use'` 

