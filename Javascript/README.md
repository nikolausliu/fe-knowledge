
# 原型及原型链

# 作用域及闭包

# Event Loop
浏览器环境和node环境下的Event Loop模型是不同的。

## 浏览器环境

1. 同步代码在执行栈中顺序执行，当遇到异步代码时，把它移入`Event Table`中注册相应的函数，然后继续执行后面的同步代码。
2. 当某个异步事件完成时，`Event Table`把注册给该事件的函数移入`Event Queue（事件队列）`，其中setTimeout/setInterval会在注册函数的同时开始计时，计时结束就把函数移入事件队列。
3. 执行栈内的同步任务执行完成处于闲置状态后，就会去`Event Queue`中按照顺序读取其中的函数，进入执行栈执行。
4. 以上过程不断重复，就叫`Event Loop（事件循环）`。
5. 其中第3步中提到的`Event Queue`又分为`macrotask（宏任务）`和`microtask（微任务）`。
  - macrotask:
    - setTimeout
    - setInterval
    - setImmediate
    - requestAnimationFrame
    - I/O
    - UI rendering
  - microtask: 
    - Promises
    - process.nextTick
    - Object.observe
    - MutationObserver
6. 执行栈内的第一层同步代码也属于宏任务，宏任务执行完后马上按照顺序执行微任务队列中的所有任务，微任务也执行完后，第一轮事件循环就结束了。继续执行下一个宏任务，此时开始了第二轮事件循环。
7. 概括一下事件循环的过程：执行完宏任务，接着执行所有的微任务。然后接着执行下一个宏任务，再接着执行所有的微任务，如此循环往复。

### 例子
```javascript
setTimeout(function(){
    console.log(1)
},0);
new Promise(function(resolve){
    console.log(2)
    for( var i=100000 ; i>0 ; i-- ){
        i==1 && resolve()
    }
    console.log(3)
}).then(function(){
    console.log(4)
});
console.log(5);
```
上面这个例子的执行顺序：

1. 执行栈执行到setTimeout，识别为异步任务，移入到Event Table并注册函数console.log(1)，同时（移入Event Table的哪一刻开始计时，0ms）把函数移入宏任务队列。
2. new Promise入栈，遇到同步代码console.log(2)，输出2。
3. 执行for循环，仍然是同步任务，遇到resolve()，识别为异步任务，移入到Event Table并注册then方法里的函数，同时把函数移入微任务队列。
4. 执行console.log(3)，输出3。new Promise出栈。
5. 执行console.log(5)，输出5。
6. 执行栈为空，开始执行微任务队列里的第一个任务。console.log(4)入栈，输出4，console.log(4)出栈。
7. 执行栈为空，第一轮事件循环结束。开始下一轮事件循环，执行宏任务队列里的第一个任务。console.log(1)入栈，输出1，console.log(1)出栈。
8. 执行栈为空，第二轮事件循环结束。

综上，打印顺序是：2,3,5,4,1。

### requestAnimationFrame
> 通常浏览器以每秒60帧（60fps）的速率刷新页面，据说这个帧率最适合人眼交互，大概16.7ms渲染一帧，所以如果要让用户觉得顺畅，单个macrotask及它相关的所有microtask最好能在16.7ms内完成。

> 但也不是每轮事件循环都会执行视图更新，浏览器有自己的优化策略，例如把几次的视图更新累积到一起重绘，重绘之前会通知requestAnimationFrame执行回调函数，也就是说requestAnimationFrame回调的执行时机是在一次或多次事件循环的UI render阶段

```javascript
setTimeout(function() {console.log('timer1')}, 0)

requestAnimationFrame(function(){
	console.log('requestAnimationFrame')
})

setTimeout(function() {console.log('timer2')}, 0)

new Promise(function executor(resolve) {
	console.log('promise 1')
	resolve()
	console.log('promise 2')
}).then(function() {
	console.log('promise then')
})

console.log('end')
```

上面代码的输出结果是不确定的。可能是`promise1, promise2, end, promise then, requestAnimationFrame, timer1, timer2`，也有可能是`promise1, promise2, end, promise then, timer1, requestAnimationFrame, timer2`，还有可能是`promise1, promise2, end, promise then, timer1, timer2, requestAnimationFrame`。共同点就是，requestAnimationFrame总是在一轮事件循环的末尾被调用，但是在哪一轮事件循环的末尾被调用就不确定了。


## node环境
```javascript
setTimeout(function() {
    console.log(1);
    new Promise(function(resolve) {
        console.log(2);
        resolve();
    }).then(function() {
        console.log(3)
    })
})

setTimeout(function() {
    console.log(4);
    new Promise(function(resolve) {
        console.log(5);
        resolve();
    }).then(function() {
        console.log(6)
    })
})
```

同样的代码，在Chrome下输出1,2,3,4,5,6。在node环境下，有时输出1,2,3,4,5,6；有时输出1,2,4,5,3,6。ummmm~~~。造成这个问题的原因是chrome和node对于Promise的实现是不同的，可以参考[这个问题下方应杭的回答](https://www.zhihu.com/question/272286497)。

### node环境下宏任务和微任务详解
1. node环境下，宏任务是分阶段的，按阶段先后执行：

- expired timers and intervals，即到期的setTimeout/setInterval
- I/O events，包含文件，网络等等
- immediates，通过setImmediate注册的函数
- close handlers，close事件的回调，比如TCP连接断开

什么意思呢？node把宏任务分成了上面4类，如果在同一轮事件循环里往宏任务里分别添加了上面4类宏任务，会按照以上顺序分别执行。

```javascript
setImmediate(() => {
    console.log(1)
})
setTimeout(() => {
	console.log(0)	
})
```

2. 微任务也分为两种，**process.nextTick**和**其它微任务**，比如在同一轮事件循环里有process.nextTick和promise，那么总是那个process.nextTick先执行。

```javascript
Promise.resolve().then(()=>{
    console.log(1)
})
process.nextTick(()=>{
    console.log(2)
})
console.log(3)
```

以上代码输出结果为3,2,1。

## 参考：
- [这一次，彻底弄懂 JavaScript 执行机制](https://juejin.im/post/59e85eebf265da430d571f89)
- [通过microtasks和macrotasks看JavaScript异步任务执行顺序](https://tuobaye.com/2017/10/24/%E9%80%9A%E8%BF%87microtasks%E5%92%8Cmacrotasks%E7%9C%8BJavaScript%E5%BC%82%E6%AD%A5%E4%BB%BB%E5%8A%A1%E6%89%A7%E8%A1%8C%E9%A1%BA%E5%BA%8F/)
- [图解搞懂JavaScript引擎Event Loop](https://juejin.im/post/5a6309f76fb9a01cab2858b1)
- [面试之Event Loop，nextTick()和setImmediate()区别分析](https://zhuanlan.zhihu.com/p/33090541)
- [nodejs中的event loop](https://www.jianshu.com/p/deedcbf68880)
- [深入理解js事件循环机制（浏览器篇）](http://lynnelv.github.io/js-event-loop-browser)
- [深入理解js事件循环机制（Node.js篇）](http://lynnelv.github.io/js-event-loop-nodejs)
- [HTML规范](https://html.spec.whatwg.org/multipage/webappapis.html#event-loop-processing-model)
- [nodejs官方文档](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

# new一个对象的过程发生了什么
1. 创建一个新对象
2. 将构造函数的作用域赋给新对象
3. 执行构造函数内的代码
4. 返回新对象

# Promise
[Promise 必知必会（十道题）](https://zhuanlan.zhihu.com/p/30797777)

# 面向对象
## Object.defineProperty
对象的属性分为**数据属性**和**访问器属性**，我们平时接触比较多的是数据属性

### 数据属性
我们随便写一个对象`var person = {name: 'xiaoMing'}`，可以看到它有一个属性name，这个name有一个具体的字符串值xiaoMing。为了描述这个属性name，我们需要用到**特性**这个概念。我们用4个特性来描述一个数据属性，为了区别属性，把特性用两个中括号括起来：

- [[configurable]]: 是否可配置。这个特性可以想象成一个总开关，如果把这个特性设置为false，那么属性就不能被更改了：不能用delete操作符操作来删除属性，不能设置属性的其它特性，也不能把数据属性修改为访问器属性。并且这个开关一旦关了就再也打不开了，也就是说一旦设置为false,就不能再设置回true了。
- [[enumerable]]: 是否可枚举。确切的说是属性能否被for-in循环枚举。
- [[writable]]: 是否可写。是否能够修改属性的值，如果设为false，那么属性就是只读的，修改属性的操作无效。
- [[value]]: 值。比如这里的name的这个特性就是字符串xiaoMing。

对于没有手动设置过特性的属性，configurable、enumerable和writable这3个特性默认都是true。想要手动修改属性的特性，需要用到`Object.defineProperty(object, prop, descriptor)`方法。

这个方法的3个参数分别是：对象（在这里就是person）、属性（字符串name）以及一个描述特性的描述符对象（比如`{writable: false, name: 'xiaoHong'}`就表示把person对象改为name值为xiaoHong，并且name不可修改）。

如果对某个对象的某个属性调用了这个方法，但是没有指定configurable、enumerable和writable这3个特性，那么这些特性就会默认被修改为false。言下之意，如果调用了这个方法，修改了person对象的name属性，但是没有设置configurable，那以后都不能调用这个方法来修改name属性了，因为开关已经关了。

`Object.defineProperties(object, descriptor)`方法可以用来一次定义多个属性的特性，第一个参数还是对象，第二个参数是`{属性：描述符对象}`这样的对象。还是以person对象为例:`Object.defineProperties(person, {name: {value: 'xiaoMing'}, age: {writable: false, value: 1}})`。

`Object.getOwnPropertyDescriptor(object, prop)`用来获取某个对象的某个属性的描述符对象。

### 访问器属性
访问器属性不包含具体的值，只包含一对getter和setter函数。数据属性要访问和设置某个属性的值很简单：`person.name`可以获取值，`person.name = 'xiaoMing'`可以设置值，访问器属性可不行。比如还是person对象，加入它有一个访问器属性age，你要获取值的时候这么写`person.age`实际上是调用了它的getter函数，函数的返回值就是`person.name`的值；你要设置值的时候写`person.name = 'xiaoMing'`实际上是把xiaoMing传给了它的setter函数，setter函数说怎么干就怎么干。

访问器属性的特性：

- [[configurable]]: 
- [[enumerable]]: 
- [[get]]: 读取属性时调用的函数，默认为undefined。
- [[set]]: 设置属性时调用的函数，默认为undefined。

访问器属性的特性也用`Object.defineProperty()`来修改。


## 创建对象
### 工厂模式
工厂模式创建的对象无法无法识别其类型。

### 构造函数模式
构造函数模式创建的对象，如果对象的方法体写在构造函数内部，则没创建一个新对象，都要为之创建一个Functions实例（定义function实际是new Function(){}的过程），浪费内存。如果把方法定义写在全局，则造成全局污染，且封装性差。

### 原型模式
原型模式创建的对象，如果原型中有引用类型的属性，那么一个实例的改属性变化会引起所有实例的变化。

### 组合使用构造函数模式和原型模式

```javascript
function Animal(name){
    this.name = name;
}
Animal.prototype.eat = function(){
    console.log(`${this.name} is eating!`);
}
```

## 原型继承
```javascript
function Animal(name) {
    this.name = name;
}
Animal.prototype.eat = function(){
    console.log(`${this.name} is eating!`);
}

// 继承属性
var Dog = function(name){
    Animal.call(this, name);
    this.legs = 4;
}

// 借用空函数
function F(){}
F.prototype = Animal.prototype;
// 继承原型
Dog.prototype = new F();
// 上一步切断了Dog.prototype与原配prototype之间的联系，constructor变了，要把constructor改回来
Dog.prototype.constructor = Dog;
// 给dog添加方法
Dog.prototype.bark = function(){
    console.log('汪汪汪!');
}

var dog = new Dog('哈士奇');
console.log(dog.name, dog.legs);    // 哈士奇  4
dog.eat();  // 哈士奇 is eating!
dog.bark(); // 汪汪汪!
```

继承这个动作可以被封装起来：

```javascript
function inherits(Child, Parent) {
    var F = function(){}
    F.prototype = Parent.prototype
    Child.prototype = new F()
    Child.prototype.constructor = Child
}
```

用上面封装的函数改写一下：

```javascript
function Animal(name) {
    this.name = name;
}
Animal.prototype.eat = function(){
    console.log(`${this.name} is eating!`);
}

function Dog(name) {
    Animal.call(this, name);
    this.legs = 4;
}
inherits(Dog, Animal);
Dog.prototype.bark = function(){
    console.log('汪汪汪!');
}

var dog = new Dog('哈士奇');
console.log(dog.name, dog.legs);    // 哈士奇  4
dog.eat();  // 哈士奇 is eating!
dog.bark(); // 汪汪汪!

```

### class继承

```javascript
class Animal {
    constructor(name) {
        this.name = name
    }
    eat(){
        console.log(`${this.name} is eating!`)
    }
}

var animal = new Animal('animal')

class Dog extends Animal {
    constructor(name) {
        super(name)
        this.legs = 4
    }
    bark() {
        console.log('汪汪汪!')
    }
}
var dog = new Dog('哈士奇');
console.log(dog.name, dog.legs);    // 哈士奇  4
dog.eat();  // 哈士奇 is eating!
dog.bark(); // 汪汪汪!
```

## 参考
- [廖雪峰的JavaScript教程](https://www.liaoxuefeng.com/wiki/001434446689867b27157e896e74d51a89c25cc8b43bdb3000/001434499763408e24c210985d34edcabbca944b4239e20000)
- 《JavaScript高级程序设计（第三版）》第六章

# Ajax

# 函数式编程

# 设计模式

# RESTful API

# Vue
## 响应式原理

# 防抖和节流



# window.RequestAnimationFrame

[MDN上的定义](https://developer.mozilla.org/zh-CN/docs/Web/API/Window/requestAnimationFrame)
- 回调会在浏览器重绘之前调用
- 回调有一个参数，当前被`requestAnimationFrame()`排序的回调函数被触发的时间
- 返回值：一个整数标志，类似于setTimeout的返回值，可以传入`window.cancelAnimationFrame()`以取消回调
- 这个方法运行一次，只执行一次动画，要生成连续的动画，需要在回调里递归的调用这个方法。
- 页面后台运行时，`requestAnimationFrame()`会暂停调用以提升性能和电池寿命。但是我在demo中发现，setTimeout的动画在后台运行时，好像也会暂停，可以看下面给出的demo。

## demo
提示：可以在页面动画加载一半的时候，切换到其他标签页，过一会再切回来，发现动画是从刚才的位置继续进行的，说明是暂停了的。
- [requestAnimationFrame的动画](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/requestAnimationFrame.html)
- [setTimeout的动画](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/setTimeoutAnimate.html)

# Element.getBoundingClientRect
[MDN上的定义](https://developer.mozilla.org/zh-CN/docs/Web/API/Element/getBoundingClientRect)
该方法返回元素的大小及其相对于视口的位置。返回值是一个DOMRect对象：`{top,right,bottom,left,width,height,x,y}`。

# Page Visibility API

[MDN上的定义](https://developer.mozilla.org/zh-CN/docs/Web/API/Page_Visibility_API)

- 页面可见性API，通过`document.addEventListener('visibilitychange')`监听页面可见性，在回调中判断`document.hidden == true | false`或者判断`document.visibilityState == 'visible' | 'hidden' | 'prerender' | 'unloaded'`。
- 使用场景：切换页面时，暂停视频或音乐，暂停轮播图自动轮播等。总结起来就是状态会随时间改变，我们希望这种状态在我们不观察时保持不变，并且这种状态可控时都能用这个API来做限制。
- 页面可见性 API对于节省资源和提高性能特别有用，它使页面在文档不可见时避免执行不必要的任务。
- `<iframe>`的可见性状态与父文档相同。使用CSS属性（例如display: none;）隐藏`<iframe>`不会触发可见性事件或更改框架中包含的文档的状态。
- "当浏览器最小化时，不会触发visibilitychange 事件，也不会设置hidden为true。"这是mdn上的原话，但是我在测试时发现浏览器最小化是会触发visibilitychange事件的，页面销毁时也会触发。

# Notifications API

<details>
<summary>查看详情</summary>

[MDN上的定义](https://developer.mozilla.org/zh-CN/docs/Web/API/notification)

## 语法：`let notification = new Notification(title, options = {dir, lang, body, tag, icon})`

- title: 标题
- dir: 文字方向 auto | ltr | rtl
- lang: 语言
- body: 标题下方显示的内容
- tag: 标签
- icon: 通知左侧图标url

## 静态属性：
- `Notification.permission`: 一个字符串，用来表示用户是否同意你在当前站点显示Notification通知。用户未操作之前，默认是值是default(浏览器当作拒绝来对待)；用户允许，该值变为granted；用户拒绝，该值变为denied。只有值为granted时，才能成功显示Notification，当然了，这个值是只读的。所以在`new Notification`之前要先判断该值是否为granted，如果不是需要向用户请求权限（`Notification.requestPermission`）。

## 静态方法：
- `Notification.requestPermission`: 这个方法接受回调函数，同时自身调用的结果又返回一个promise，传入两者中的参数都是permission。

## 实例方法：
- `notification.close()`: 关闭现在正在显示的Notification通知

## 事件处理：
这个比较常用的应该是`onclick`，比如点击通知跳转到某个页面去。
- `Notification.onshow`
- `Notification.onclick`
- `Notification.onclose`
- `Notification.onerror`

## 注意：
- title必传，但是可以传空字符串，options非必传。它们都有默认值
- dir我设置了好像并没有看到效果
- body接受换行
- 如果后续通知用了之前用过的tag就不会显示了，暂时不清楚这个在同一个站点下有没有时间限制
- 在各浏览器下的UI和行为不同。比如同样在百度站点下`new Notification('')`Firefox在title处什么都不显示，点击通知会跳转回Firefox调用Notification的那个页面。微软的Edge也会跳转，但是title会显示百度的域名。Google Chrome会在title处显示Google Chrome，但是点击后会直接关闭而不跳转。

## Demo:
- [demo](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/notification.html)
- [promise-demo](https://htmlpreview.github.io/?https://github.com/nikolausliu/fe-knowledge/blob/master/Javascript/notification-promise.html)

</details>


# 面试题
- [记一道经典前端题](https://juejin.im/post/5c6cb5bae51d45012d068579)
- [深拷贝](https://juejin.im/post/5c45112e6fb9a04a027aa8fe)