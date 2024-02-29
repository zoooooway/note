# React 学习笔记
> 参阅: [React官方网站](https://zh-hans.react.dev/learn) 

## 语法

### JSX
JSX 是 JavaScript 语法扩展，可以让你在 JavaScript 文件中书写类似 HTML 的标签。
JSX 看起来和 HTML 很像，但它的语法更加严格并且可以动态展示信息。

#### JSX 规则:
1. 只能返回一个根元素 
   如果想要在一个组件中包含多个元素，需要用一个父标签把它们包裹起来。
   如果你不想在标签中增加一个额外的 `<div>`，可以用 `<>` 和 `</>` 元素来代替。这个空标签被称作 Fragment。React Fragment 允许你将子元素分组，而不会在 HTML 结构中添加额外节点。

   > JSX 虽然看起来很像 HTML，但在底层其实被转化为了 JavaScript 对象，你不能在一个函数中返回多个对象，除非用一个数组把他们包装起来。
   > 这就是为什么多个 JSX 标签必须要用一个父元素或者 Fragment 来包裹。

2. 标签必须闭合
   JSX 要求标签必须正确闭合。像 `<img>` 这样的自闭合标签必须书写成 `<img />`，而像 `<li>oranges` 这样只有开始标签的元素必须带有闭合标签，需要改为 `<li>oranges</li>`。

3. 使用驼峰式命名法给大部分属性命名
   JavaScript 对变量的命名有限制。例如，变量名称不能包含 - 符号或者像 class 这样的保留字。由于 class 是一个保留字，所以在 React 中需要用 className 来代替。
   由于历史原因，aria-* 和 data-* 属性是以带 - 符号的 HTML 格式书写的。

#### 传递属性
当你想把一个字符串属性传递给 JSX 时，把它放到单引号或双引号中即可。
当需要传递变量时, 用 { 和 } 替代 " 和 " 以使用 JavaScript 变量

在 JSX 中，只能在以下两种场景中使用大括号：
* 用作 JSX 标签内的文本：`<h1>{name}'s To Do List</h1>` 是有效的，但是 `<{tag}>Gregorio Y. Zara's To Do List</{tag}>` 无效。
* 用作紧跟在 = 符号后的 属性：src={avatar} 会读取 avatar 变量，但是 src="{avatar}" 只会传一个字符串 {avatar}。

可以在 JSX 中传递对象, 对象也用大括号表示。为了能在 JSX 中传递，你必须用另一对额外的大括号包裹对象：`person={{ name: "Hedy Lamarr", inventions: 5 }}`。

#### Props


## 框架

### React Router