---
layout:     post
title:      "01 Stack and Queue"
subtitle:   "chapter 2 of Princeton Algorithm"
date:       2021-03-14
author:     "Miley"
header-img: "img/post-bg-rwd.jpg"
tags:
    - Data Structures
    - Java
---



## Catalog


1.  [Linked List Implementation](#利用链表实现栈)
2.  [Resizing Array Implementation](#利用扩容数组实现栈)
3.  [RequireJS & AMD](#requirejs--amd)
4.  [SeaJS & CMD](#seajs--cmd)


> Stack: Examine the item most reccently added ("last in forst out").

## 利用链表实现栈

```js
public class Stack<Item>
{
    private Node first;
    private int N;
    //inner class
    private class Node
    {
        Item item;
        Node next;
    }
    
    public boolean isEmpty() 
    {return first == null;}
    
    public int size()
    { return N; }
    
    public void push(Item item){
        Node oldFirst = first;
        first = new Node();
        first.item = item;
        first.next = oldFirst;
        N++;
    }
    
    public Item pop(){
        Item item = first.item;
        first = first.next;
        N--;
        return item;
    }
}
```
一个有N个元素的Stack,每个元素占用的内存：  
* 16 bytes for class overhead `Stack`  
* 8 bytes for inner class extra overhead. As the inner class `Node` is not static, so it has a reference to the outer class.  
* 8 bytes for the reference to String
* 8 bytesfor the reference to Node
Total: 40 bytes per stack node
ref: [Java:Size of inner class-Stackoverflow](https://stackoverflow.com/questions/12193116/java-size-of-inner-class)  

### More:
#### 1.1
new Object()将占用多少byte的内存空间？
```js
原生类型(primitive type)的内存占用
Primitive Type      Memory Required(bytes)
    boolean                      1
    byte                         1
    short                        2
    char                         2
    int                          4
    float                        4
    long                         8
    double                       8
```
#### 1.2
静态类与非静态类
In **static** method, The memory of a static method is fixed in the ram, for this reason we don’t need the object of a class in which the static method is defined to call the static method. To call the method we need to write the name of the method followed by the class name.   
```js
class GFG{
 public static void geek()
 { }
}
// calling
GFG.geek();

```
In **non-static** method, the memory of non-static method is not fixed in the ram, so we need class object to call a non-static method. To call the method we need to write the name of the method followed by the class object name.
```js
class GFG{
 public void geek()
 { }
}
// creating object
GFG g = new GFG();
// calling
g.geek();
```

## 利用扩容数组实现栈
增加了resize函数：
```js
public class Stack<Item>
{
    private Item[] s;
    private int N = 0;
    
    public Stack(int capacity)
    {s = (Item[]) new Object[capacity];}
    
    public boolean isEmpty() 
    {return N == 0;}
    
    public int size()
    {return N;}
    
    private void resize(int capacity){
        Item[] copy = (Item[]) new Object[capacity];
        for(int i = 0; i < s.length; i++){
            copy[i] = s[i];
        }
        s = copy;
    }
    
    public void push(Item item){
        if(N == s.length) resize(2 * s.length)
        s[N++] = item;
    }
   
    public Item pop(){
        Item item = s[--N];
        s[N] = null; // avoid loitering
        if(N > 0 && N == s.length/4) resize(s.length/2);
        return item;
    }
}
```
### More
#### 2.1
what is loitering?
> Loitering: Holding a reference to an object when it is no longer needed. -- _Algorithm_, Priceton Universily  

example:
```js
// loitering
public String pop()
{ return s[--N]; }

// avoid loitering
public String pop()
{
 String item = s[--N];
 s[N] = null;
 return item;
} 
```

## RequireJS  AMD

[AMD (Async Module Definition)](http://wiki.commonjs.org/wiki/Modules/AsynchronousDefinition) 是 RequireJS 在推广过程中对模块定义的规范化产出。

> RequireJS is a JavaScript file and module loader. It is optimized for in-browser use, but it can be used in other JavaScript environments

RequireJS 主要解决的还是 CommonJS 同步加载脚本不适合浏览器 这个问题：

```js
//CommonJS

var Employee = require("types/Employee");

function Programmer (){
    //do something
}  

Programmer.prototype = new Employee();

//如果 require call 是异步的，那么肯定 error
//因为在执行这句前 Employee 模块肯定来不及加载进来
```
> As the comment indicates above, if require() is async, this code will not work. However, loading scripts synchronously in the browser kills performance. So, what to do?

所以我们需要 **Function Wrapping** 来获取依赖并且提前通过 script tag 提前加载进来


```js
//AMD Wrapper

define(
    [types/Employee],    //依赖
    function(Employee){  //这个回调会在所有依赖都被加载后才执行

        function Programmer(){
            //do something
        };

        Programmer.prototype = new Employee();
        return Programmer;  //return Constructor
    }
)
```

当依赖模块非常多时，这种**依赖前置**的写法会显得有点奇怪，所以 AMD 给了一个语法糖， **simplified CommonJS wrapping**，借鉴了 CommonJS 的 require 就近风格，也更方便对 CommonJS 模块的兼容：

```js
define(function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');

    return function () {};
});
```
The AMD loader will parse out the `require('')` calls by using `Function.prototype.toString()`, then internally convert the above define call into this:

```js
define(['require', 'dependency1', 'dependency2'], function (require) {
    var dependency1 = require('dependency1'),
        dependency2 = require('dependency2');

    return function () {};
});
```

出于`Function.prototype.toString()`兼容性和性能的考虑，最好的做法还是做一次 **optimized build**



AMD 和 CommonJS 的核心争议如下：

### 1. **执行时机**

Modules/1.0:

```js
var a = require("./a") // 执行到此时，a.js 才同步下载并执行
```

AMD: （使用 require 的语法糖时）

```js
define(["require"],function(require)){
    // 在这里，a.js 已经下载并且执行好了
    // 使用 require() 并不是 AMD 的推荐写法
    var a = require("./a") // 此处仅仅是取模块 a 的 exports
})
```

AMD 里提前下载 a.js 是出于对浏览器环境的考虑，只能采取异步下载，这个社区都认可（Sea.js 也是这么做的）

但是 AMD 的执行是 Early Executing，而 Modules/1.0 是第一次 require 时才执行。这个差异很多人不能接受，包括持 Modules/2.0 观点的人也不能接受。

### 2. **书写风格**

AMD 推荐的风格并不使用`require`，而是通过参数传入，破坏了**依赖就近**：

```js
define(["a", "b", "c"],function(a, b, c){
    // 提前申明了并初始化了所有模块

    true || b.foo(); //即便根本没用到模块 b，但 b 还是提前执行了。
})
```

不过，在笔者看来，风格喜好因人而异，主要还是**预执行**和**懒执行**的差异。

另外，require 2.0 也开始思考异步处理**软依赖**（区别于一定需要的**硬依赖**）的问题，提出了这样的方案：

```js
// 函数体内：
if(status){
    async(['a'],function(a){
        a.doSomething()
    })
}
```

## SeaJS & CMD

CMD (Common Module Definition) 是 [SeaJS](http://seajs.org/docs/) 在推广过程中对模块定义的规范化产出，是 Modules/2.0 流派的支持者，因此 SeaJS 的模块写法尽可能与 Modules/1.x 规范保持一致。

不过目前国外的该流派都死得差不多了，RequireJS 目前成为浏览器端模块的事实标准，国内最有名气的就是玉伯的 Sea.js ，不过对国际的推广力度不够。

* CMD Specification
    * [English (CMDJS-repo)](https://github.com/cmdjs/specification/blob/master/draft/module.md)
    * [Chinese (SeaJS-repo)](https://github.com/seajs/seajs/issues/242)


CMD 主要有 define, factory, require, export 这么几个东西

 * define `define(id?, deps?, factory)`
 * factory `factory(require, exports, module)`
 * require `require(id)`
 * exports `Object`


CMD 推荐的 Code Style 是使用 CommonJS 风格的 `require`：

* 这个 require 实际上是一个全局函数，用于加载模块，这里实际就是传入而已

```js
define(function(require, exports) {

    // 获取模块 a 的接口
    var a = require('./a');
    // 调用模块 a 的方法
    a.doSomething();

    // 对外提供 foo 属性
    exports.foo = 'bar';
    // 对外提供 doSomething 方法
    exports.doSomething = function() {};

});
```

但是你也可以使用 AMD 风格，或者使用 return 来进行模块暴露

```js
define('hello', ['jquery'], function(require, exports, module) {

    // 模块代码...

    // 直接通过 return 暴露接口
    return {
        foo: 'bar',
        doSomething: function() {}
    };

});
```



Sea.js 借鉴了 RequireJS 的不少东西，比如将 FlyScript 中的 module.declare 改名为 define 等。Sea.js 更多地来自 Modules/2.0 的观点，但尽可能去掉了学院派的东西，加入了不少实战派的理念。
