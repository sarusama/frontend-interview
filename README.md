# frontend-interview

frontend interview

## es5，es6，es7的区别

es5 扩展了Object、Array、Function等的功能

es6 类、模块化、箭头函数、函数参数默认值、模版字符串、解构赋值、延展操作符(...)、对象属性简写（对象扩展，还有计算属性）、Promise、let、const（块级作用域）、新数据结构Set、Map、Symbol

es7 Array.prototype.includes()、指数操作符

es8 async/await、Object.values()、Object.entries()、String padding、函数参数列表结尾允许逗号、Object.getOwnPropertyDescriptors()等

## JS实现继承

通过Object.create来划分为两类继承方式，每类继承方式又有两种继承方式。不用Object.create的 -- 原型链继承方式和构造函数继承方式，他们都有各自的优缺点为了弥补各自的优缺点衍生出了组合继承方式；使用Object.create -- 原型式继承方式，寄生式继承方式，他们也有各自缺点，也进行组合成为寄生式组合继承方式。最后的寄生式组合继承方式是通过组合继承改造之后的最优继承方式，而 extends 的语法糖和寄生组合继承的方式基本类似

### 原型链继承

创建的实例使用的是同一个原型对象，内存空间是共享的
```
function Parent() {
    this.name = 'name';
    this.play = [1, 2, 3];
}
function Child() {
    this.type = 'child';
}
Child.prototype = new Parent();
console.log(new Child());
var s1 = new Child();
var s2 = new Child();
s1.play.push(4);
console.log(s1.play, s2.play); // [1, 2, 3, 4] [1, 2, 3, 4]
```

### 构造函数继承（借助 call）

相比原型链继承方式，父类的引用属性不会被共享，优化了原型链继承方式的弊端，但是只能继承父类的实例属性和方法，不能继承原型属性或者方法
```
function Parent(){
    this.name = 'name';
}

Parent.prototype.getName = function () {
    return this.name;
}

function Child(){
    Parent.call(this);
    this.type = 'child'
}

let child = new Child();
console.log(child);  // 没问题
console.log(child.getName());  // 会报错
```

### 组合继承
解决了原型链继承方式和构造函数继承方式的问题，但是造成了多构造一次的性能开销
```
function Parent () {
    this.name = 'name';
    this.play = [1, 2, 3];
}

Parent.prototype.getName = function () {
    return this.name;
}
function Child() {
    // 第二次调用 Parent()
    Parent.call(this);
    this.type = 'child';
}

// 第一次调用 Parent()
Child.prototype = new Parent();
// 手动挂上构造器，指向自己的构造函数
Child.prototype.constructor = Child;
var s3 = new Child();
var s4 = new Child();
s3.play.push(4);
console.log(s3.play, s4.play);  // 不互相影响
console.log(s3.getName()); // 正常输出'parent3'
console.log(s4.getName()); // 正常输出'parent3'
```

### 原型式继承
这种继承方式的缺点也很明显，因为Object.create方法实现的是浅拷贝，多个实例的引用类型属性指向相同的内存，存在篡改的可能
```
let parent = {
    name: "name",
    friends: ["p1", "p2", "p3"],
    getName: function() {
        return this.name;
    }
};

  let person1 = Object.create(parent);
  person1.name = "tom";
  person1.friends.push("jerry");

  let person2 = Object.create(parent);
  person2.friends.push("lucy");

  console.log(person1.name); // tom
  console.log(person1.name === person1.getName()); // true
  console.log(person2.name); // name
  console.log(person1.friends); // ["p1", "p2", "p3","jerry","lucy"]
  console.log(person2.friends); // ["p1", "p2", "p3","jerry","lucy"]
```

### 寄生式继承
寄生式继承在上面继承基础上进行优化，利用这个浅拷贝的能力再进行增强，添加一些方法。其优缺点也很明显，跟原型式继承一样
```
let parent = {
    name: "name",
    friends: ["p1", "p2", "p3"],
    getName: function() {
        return this.name;
    }
};

function clone(original) {
    let clone = Object.create(original);
    clone.getFriends = function() {
        return this.friends;
    };
    return clone;
}

let person = clone(parent);

console.log(person.getName()); // name
console.log(person.getFriends()); // ["p1", "p2", "p3"]
```

### 寄生组合式继承
寄生组合式继承，借助解决普通对象的继承问题的Object.create 方法，在亲全面几种继承方式的优缺点基础上进行改造，这也是所有继承方式里面相对最优的继承方式。extends也是用寄生组合式继承。
```
function clone (parent, child) {
    // 这里改用 Object.create 就可以减少组合继承中多进行一次构造的过程
    child.prototype = Object.create(parent.prototype);
    child.prototype.constructor = child;
}

function Parent() {
    this.name = 'name';
    this.play = [1, 2, 3];
}
Parent.prototype.getName = function () {
    return this.name;
}
function Child() {
    Parent.call(this);
    this.friends = 'child5';
}

clone(Parent, Child);

Child.prototype.getFriends = function () {
    return this.friends;
}

let person = new Child(); 
console.log(person); //{friends:"child5",name:"child5",play:[1,2,3],__proto__:Parent6}
console.log(person.getName()); // parent6
console.log(person.getFriends()); // child5
```

## :伪类，::伪元素

## css3有哪些新增的属性

渐变，动画，变形，过渡，圆角，阴影，flexbox，word-break，text-overflow

## vue的数据绑定原理

vue2是用过Object.defineProperty实现数据响应式。Vue数据双向绑定原理是通过 数据劫持结合发布者-订阅者模式 的方式来实现的，首先是通过ES5提供的 Object.defineProperty() 方法来劫持（监听）各属性的getter、setter，并在当监听的属性发生变动时通知订阅者，是否需要更新，若更新就会执行对应的更新函数。

## vue实现动画原理

通过切换元素上的class
使用第三方动画库
使用钩子函数before-enter,enter,after-enter,before-leave,leave,after-leave操作dom

## 输入完网址敲下回车

解析url -> 浏览器缓存 -> 主机缓存 -> DNS解析 -> 获取mac地址 -> TCP三次握手 -> 发送请求 -> 响应请求 -> 渲染页面 -> 四次挥手

### 渲染页面

解析html构建dom树
解析css生成css规则树
合并 DOM 树和 CSS 规则，生成 render 树
布局 render 树（ Layout / reflow 回流 ），负责各元素尺寸、位置的计算
绘制 render 树（ paint 重绘 ），绘制页面像素信息
浏览器会将各层的信息发送给 GPU，GPU 会将各层合成（ composite ），显示在屏幕上

简单一点说就是，构建dom，加载子资源(css这些)，下载并执行js，计算样式，布局(回流)，绘制(重绘)，合成


## vue使用template和render的区别
```
createElement(标签名称, 属性配置, children)
createElement("div", {
    "class": {
        "abc": true,
        "a": this.props.a
    },
    "props": {
        "a": {
            "type": Boolean
        }
    }
})
```
1、render渲染方式可以让我们将js发挥到极致，因为render的方式其实是通过createElement()进行虚拟DOM的创建。逻辑性比较强，适合复杂的组件封装。
2、template是类似于html一样的模板来进行组件的封装。
3、render的性能比template的性能好很多
4、render函数优先级大于template
5、render函数要求更高

## 兼容浏览器

pc端浏览器不同:
html兼容问题
css兼容问题
js兼容问题

移动端和pc端兼容问题

## http协议

1xx 信息响应
2xx 成功响应
3xx 重定向
4xx 客户端响应
5xx 服务端响应

### GET请求还是POST请求

、、、
GET请求比POST请求更简单和更快，使用途径更广泛。
GET要发cookie，能被浏览器缓存，明文传参。
但是在以下情况下总是使用POST请求:
1. 发送大量数据给服务器（POST请求没有大小限制）
2. 发送用户输入的内容（包括未知字符），POST请求比GET请求更加安全
3. 需要上传文件到服务器（更新服务器上的文件或者是数据库）
、、、

## vue和react之间的区别

### 相同点
都有组件化思想
都支持服务器端渲染
都有Virtual DOM（虚拟dom）
数据驱动视图
都有支持native的方案：Vue的weex、React的React native
都有自己的构建工具：Vue的vue-cli、React的Create React App

### 区别
数据流向的不同。react从诞生开始就推崇单向数据流，而Vue是双向数据流
数据变化的实现原理不同。react使用的是不可变数据，而Vue使用的是可变的数据
组件化通信的不同。react中我们通过使用回调函数来进行通信的，而Vue中子组件向父组件传递消息有两种方式：事件和回调函数
diff算法不同。react主要使用diff队列保存需要更新哪些DOM，得到patch树，再统一操作批量更新DOM。Vue 使用双向指针，边对比，边更新DOM

## vue生命周期

beforeCreate
created
beforeUpdate
updated
beforeMount
mounted
activated
deactivated
beforeUnmount -- 原beforeDestroy
unmounted -- destroyed
errorCaptured
renderTracked 跟踪虚拟 DOM 重新渲染时调用。该事件告诉哪个操作跟踪了组件以及该操作的目标对象和键。
```
<div id="app">
  <button v-on:click="addToCart">Add to cart</button>
  <p>Cart({{ cart }})</p>
</div>
const app = createApp({
  data() {
    return {
      cart: 0
    }
  },
  renderTracked({ key, target, type }) {
    console.log({ key, target, type })
    /* 当组件第一次渲染时，这将被记录下来:
    {
      key: "cart",
      target: {
        cart: 0
      },
      type: "get"
    }
    */
  },
  methods: {
    addToCart() {
      this.cart += 1
    }
  }
})
```
renderTriggered 当虚拟 DOM 重新渲染被触发时调用。该事件告诉你是什么操作触发了重新渲染，以及该操作的目标对象和键。
```
<div id="app">
  <button v-on:click="addToCart">Add to cart</button>
  <p>Cart({{ cart }})</p>
</div>
const app = createApp({
  data() {
    return {
      cart: 0
    }
  },
  renderTriggered({ key, target, type }) {
    console.log({ key, target, type })
  },
  methods: {
    addToCart() {
      this.cart += 1
      /* 这将导致 renderTriggered 被调用
        {
          key: "cart",
          target: {
            cart: 1
          },
          type: "set"
        }
      */
    }
  }
})

app.mount('#app')
```

## 防抖 节流

防抖 - 在事件被触发n秒后再执行回调，如果在这n秒内又被触发，则重新计时。
搜索框搜索输入。只需用户最后一次输入完，再发送请求
手机号、邮箱验证输入检测
窗口大小Resize。只需窗口调整完成后，计算窗口大小。防止重复渲染。

节流 - 隔一段时间触发一次
滚动加载，加载更多或滚到底部监听
谷歌搜索框，搜索联想功能
高频点击提交，表单重复提交

## eventloop事件循环

宏任务，微任务

同步任务，异步任务

## 跨域

JSONP - callback
CORS - access-control-allow-origin
Proxy - 客户端代理 - webpack，服务端代理 - 请求转发

## 图片懒加载

将图片链接放到data-src，然后当图片出现在屏幕内，用data-src替换src。

## vue路由，组件懒加载
```
() => import("路径")
```
