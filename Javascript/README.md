--- 基本概念 ---
# 原型及原型链

# 作用域及闭包

# Event Loop

# new一个对象的过程发生了什么
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象
3. 执行构造函数内的代码
4. 返回新对象

# 面向对象

# 函数式编程

# 设计模式

# RESTful API

--- Web Apis ---
# window.RequestAnimationFrame

# Element.getBoundingClientRect

# Page Visibility API

# Notification
[MDN上](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)

**语法：**`let notification = new Notification(title, options = {dir, lang, body, tag, icon})`

- title: 标题
- dir: 文字方向 auto | ltr | rtl
- lang: 语言
- body: 标题下方显示的内容
- tag: 标签
- icon: 通知左侧图标url

**静态属性：**
- `Notification.permission`: 一个字符串，用来表示用户是否同意你在当前站点()显示Notification通知。用户未操作之前，默认是值是default(浏览器当作拒绝来对待)；用户允许，该值变为granted；用户拒绝，该值变为denied。只有值为granted时，才能成功显示Notification，当然了，这个值是只读的。所以在`new Notification`之前要先判断该值是否为granted，如果不是需要向用户请求权限（`Notification.requestPermission`）。

**静态方法：**
- `Notification.requestPermission`: 这个方法接受回调函数，同时自身调用的结果又返回一个promise，传入两者中的参数都是permission。

**实例方法：**
- `notification.close()`: 关闭现在正在显示的Notification通知

**事件处理：**
这个比较常用的应该是`onclick`，比如点击通知跳转到某个页面去。
- `Notification.onshow`
- `Notification.onclick`
- `Notification.onclose`
- `Notification.onerror`

**注意：**
- title必传，但是可以传空字符串，options非必传。它们都有默认值
- dir我设置了好像并没有看到效果
- body接受换行
- 如果后续通知用了之前用过的tag就不会显示了，暂时不清楚这个在同一个站点下有没有时间限制
- 在各浏览器下的UI和行为不同。比如同样在百度站点下`new Notification('')`Firefox在title处什么都不显示，点击通知会跳转回Firefox调用Notification的那个页面。微软的Edge也会跳转，但是title会显示百度的域名。Google Chrome会在title处显示Google Chrome，但是点击后会直接关闭而不跳转。

**Demo:**
[demo](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/notification.html)
[promise-demo](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/notification-promise.html)
