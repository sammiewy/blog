---
title: ES6中的Generator函数
date: 2016-06-30 20:10:10
tags: ES6
categories: ES6
---

#### 一、ES6简介
 ECMAScript 6（以下简称 ES6）是 JavaScript 语言的下一代标准，已经在 2015 年 6 月正式发布了。
 ES6中包含了许多新的语言特性，它们将使JS变得更加强大，更富表现力。

##### 1、现状
- 主流框架全面转向 ES6

    [Angular 2](https://github.com/angular/angular)

    [ReactJs](http://reactjs.cn/)

    [koa](https://github.com/koajs/koa)
- 兼容性 [对比表格](http://kangax.github.io/compat-table/es6/)

##### 2、转码器
转码器可以将ES6代码转为ES5代码，常用的转码器：[babel转码器](https://babeljs.io/)、
[Traceur转码器](https://github.com/google/traceur-compiler) 、 [Traceur在线转码](http://google.github.io/traceur-compiler/demo/repl.html)

##### 3、新特性
- let and const命令
-   Arrow Function箭头函数
-  Destructuring Assignment （解构赋值）
-  Generator函数
-  Class
-  Module
-  Promise

更多可参考[es6-features](http://es6-features.org/#Constants)

#### 二、Generator函数
在讲Generator之前，我们来说一下我们可能会遇到的这种场景：

```
// 第1个ajax请求
$.ajax({
    url: '/bankend/xxx/xxx',
    dateType:'json',
    type:'get',
    data:{
        data:JSON.stringify({status:1,data:'hello world'}),
        type:'json',
        timeout:1000
    },
    success:function(data){
        if(data.status === 1){
            // 第2个ajax请求
            $.ajax({
                ......此处省略500字
                success:function(data){
                    if(data.status === 1){
                        // 第3个ajax请求
                        $.ajax({
                            ......此处省略500字
                            success:function(data){
                                if(data.status === 1){

                                }
                            }
                        });
                    }
                }
            });
        }
    }
});
```
如果改用generator函数实现方式：

```
yield $.ajax(...); //第一个ajax请求
yield $.ajax(...); //第二个ajax请求
yield $.ajax(...); //第三个ajax请求
......   //第n个请求 
```
这样的代码是不是让人看着更舒服一些。generator是如果实现的呢。首先我们得先了解一下generator的基本用法。

##### 1、基本语法
Generator函数是ES6中的一种异步实现函数，它类似一种状态机，通过yield产生各种状态，通过next方法执行各种状态。

基本语法如下:

```
//声明Generator函数
function* gen() {
    yield 1;
    yield 2;
    yield 3;
    yield 4;
}

//调用
var  g = gen();
g.next(); // {value: 1, done: false}
g.next(); // {value: 2, done: false}
g.next(); // {value: 3, done: false}
g.next(); // {value: 4, done: false}
g.next();  // {value: undefined, done: true}
g.next(); //{value: undefined, done: true}
```

调用generator函数返回的是一个指向内部状态的指针对象，即遍历器对象;通过返回的遍历器对象的next方法，使得指针指向下一个状态。每次调用next方法时，内部指针就从函数头或上一次停下来的地方开始执行，直到遇到下一个状态（下一条yield语句或return语句）为止。

如下图所示：

![image](http://haitao.nos.netease.com/6ad3f9d783c244d59dc785bbde15e4bd.png)

[Demo](http://people.mozilla.org/~jorendorff/demos/meow.html)

##### 2、next方法参数、yield*语句

###### （1）. next方法参数
yield语句本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数会被当作上一条yield语句的返回值。不明白啥意思，来个栗子：

```
function add(num1, num2) {
    return num1 + num2;
}

function* gen() {
    var sum = add(yield 1, yield 3);
    return sum;
}

var g = gen();
g.next(); //{ value: 1, done: false }
g.next(); //{ value: 3, done: false }
g.next(); //{ value: NaN, done: true }
g.next(); //{ value: undefined, done: true }

```

```
var g = gen();
g.next(); //{ value: 1, done: false }
g.next(2); //{ value: 3, done: false }
g.next(4); //{ value: 6, done: true }
g.next(); //{ value: undefined, done: true }

```

###### （2）. yield*语句

如果在Generator函数内部调用另外一个Generator函数，需要yield*语句。


```
function* innerGen() {
    yield 2;
    yield 3;
}

function* outGen() {
    yield 1;
    innerGen();
    yield 4;
}

function* outGenYield() {
    yield 1;
    yield* innerGen();
    yield 4;
}
```

```
var g = outGen();
g.next(); //{ value: 1, done: false }
g.next(); //{ value: 4, done: false }
g.next(); //{ value: undefined, done: true }
g.next(); //{ value: undefined, done: true }
g.next(); //{ value: undefined, done: true }


var outgen = outGenYield();
outgen.next(); //{ value: 1, done: false }
outgen.next(); //{ value: 2, done: false }
outgen.next(); //{ value: 3, done: false }
outgen.next(); //{ value: 4, done: false }
outgen.next(); //{ value: undefined, done: true }

```
使用yield*语句，generator函数的状态变化如下图所示：
![图2](http://haitao.nos.netease.com/a2d7daf768ca48ceb17eec9af02816e3.png)

看下面例子输出什么：

```
function* gen() {
    for (let i=0; i < 5; i++) {
        console.log(i);
        let temp = yield i;
        console.log(i);
        console.log(temp);
    }
}
var g = gen();
console.log(g.next()); // ?
console.log(g.next()); //?
console.log(g.next()); // ?
console.log(g.next()); // ?
console.log(g.next()); // ?
console.log(g.next()); // ?

//输出结果？
```

##### 3、for...of循环、throw、return方法
###### （1）. for...of循环
for...of循环可以自动遍历Generator函数，不需要再使用next方法。for...of只循环Generator的每个状态，不包含开始和结束节点。

```
function* gen() {
    console.log("front 1");
    yield 1;
    console.log("front 2");
    yield 2;
    console.log("front 3");
    yield 3;
    console.log("front 4");
    yield 4;
    console.log("front 5");
    return 5;
}

for (let v of gen()) {
    console.log(v);
}
// 结果
front 1
1
front 2
2
front 3
3
front 4
4
front 5

```
###### （2）. throw方法
Generator函数返回的遍历器对象都有一个throw方法， Generator.prototype.throw(),可以在函数体外抛出错误，然后在Generator函数体内捕获。

```
var g = function* () {
  try {
    yield;
  } catch (e) {
    console.log('内部捕获', e);
  }
};

var i = g();
i.next();

try {
  i.throw('a');
  i.throw('b');
} catch (e) {
  console.log('外部捕获', e);
}
// 内部捕获 a
// 外部捕获 b
```

###### （3）. return方法
Generator函数返回的遍历器对象还有一个return方法，返回给定的值，并终结Generator函数的遍历。
```
function* gen() {
    yield 1;
    yield 2;
    yield 3;
    return 4;
}

var g = gen();
g.next();  // {value: 1, done: false}
g.return(6); //{value: 6, done: true}
g.next();   //{valude: undefined, done: true}
```
#### 三、 应用
##### 1. Generator实现异步
Generator函数可以暂停执行和恢复执行，这是它能封装异步任务的根本原因。

来看一下开始的例子：

```
function* gen(){
  yield $.ajax(...); //第一个ajax请求
  yield $.ajax(...); //第二个ajax请求
  yield $.ajax(...); //第三个ajax请求
}

```
如果写成这样子我们改如何让请求一个个完成或再执行下一个请求呢。

**思路：**
当完成一个请求，我们再调用下一个next()执行下一个状态即下一个请求。

```
function next(err, data) {
    var result = gen.next(data);
    if (result.done) return;
    result.value.then(function(){
        next();
    });
}
next();
```
![image](http://haitao.nos.netease.com/8ec00addae2343d7871dcf23a1c58254.png)


co就是上面那个自动执行器的扩展。[co模块源码](https://github.com/tj/co)
用法:
```
co(function* () {
  var result = yield Promise.resolve(true);
  return result;
}).then(function (value) {
  console.log(value);
}, function (err) {
  console.error(err.stack);
});

```
##### 2. koa框架
[koa](http://koa.rednode.cn/)是由Express是Express原班人马打造的一个更小，基于nodejs平台的下一代web开发框架。Koa的精妙之处就在于其使用generator，实现了一种更为有趣的中间件系统，Koa的中间件是一系列generator函数的对象，执行起来有点类似于栈的结构，依次执行。

当一个请求过来的时候，会依次经过各个中间件进行处理，中间件跳转的信号是yield next，当到某个中间件后，该中间件处理完不执行yield next的时候，然后就会逆序执行前面那些中间件剩下的逻辑。、

总体架构图：

![image](http://haitao.nos.netease.com/916ebdb8f2db4d05a47f3d35c4b6d3a4.png)


内部实现图：
![image](http://haitao.nos.netease.com/f2044faa49d944ed91c05d4264e766ef.png)


直接上个官网的例子：

```
var koa = require('koa');
var app = koa();

// response-time中间件
app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  this.set('X-Response-Time', ms + 'ms');
});

// logger中间件
app.use(function *(next){
  var start = new Date;
  yield next;
  var ms = new Date - start;
  console.log('%s %s - %s', this.method, this.url, ms);
});

// 响应中间件
app.use(function *(){
  this.body = 'Hello World';
});
app.listen(3000);

```
上面的执行顺序就是：

请求 ==> response-time中间件 ==> logger中间件 ==> 响应中间件 ==> logger中间件 ==> response-time中间件 ==> 响应。

更详细描述就是：请求进来，先进到response-time中间件，执行 var start = new Date; 然后遇到yield next，则暂停response-time中间件的执行，跳转进logger中间件，同理，最后进入响应中间件，响应中间件中没有yield next代码，则开始逆序执行，也就是再先是回到logger中间件，执行yield next之后的代码，执行完后再回到response-time中间件执行yield next之后的代码。

koa中间的实现原理：

```
app.use = function(fn){
  if (!this.experimental) {
    // es7 async functions are not allowed,
    // so we have to make sure that `fn` is a generator function
    assert(fn && 'GeneratorFunction' == fn.constructor.name, 'app.use() requires a generator function');
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
};


app.callback = function(){
  if (this.experimental) {
    console.error('Experimental ES7 Async Function support is deprecated. Please look into Koa v2 as the middleware signature has changed.')
  }
  var fn = this.experimental
    ? compose_es7(this.middleware)
    : co.wrap(compose(this.middleware));
  var self = this;

  if (!this.listeners('error').length) this.on('error', this.onerror);

  return function(req, res){
    res.statusCode = 404;
    var ctx = self.createContext(req, res);
    onFinished(res, ctx.onerror);
    fn.call(ctx).then(function () {
      respond.call(ctx);
    }).catch(ctx.onerror);
  }
};

function compose(middleware){
  return function *(next){
    if (!next) next = noop();

    var i = middleware.length;

    while (i--) {
      next = middleware[i].call(this, next);
    }

    return yield *next;
  }
}

/**
 * Noop.
 *
 * @api private
 */

function *noop(){}
```

参考文章：

[原生httpServer、
connect、express简单介绍](https://github.com/alsotang/node-lessons/tree/master/lesson18)

[koa源码](https://github.com/koajs/koa)
