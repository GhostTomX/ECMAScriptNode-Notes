# bind()&apply&call

### 参考资料：

- [MDN](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Function/bind)
- [深入浅出妙用 Javascript 中 apply、call、bind](http://web.jobbole.com/83642/)

今天开始写js的数据结构，遇到了这个bind()的保护私有变量的用法，之前在看nodejs的http的源码也见到了bind()，因此就仔细了解了一下。

### bind()&apply&call区别

bind()与apply还有call很类似，都是Function.prototype的方法，所以就一起说一下不同点。 call和apply已经见到很多次了，在[继承](https://github.com/GhostTomX/ECMAScriptNode-Notes/blob/master/js/06_Object.md)里面，以及在讲解[this](https://github.com/GhostTomX/ECMAScriptNode-Notes/blob/master/js/07_Function_this.md)的时候也有遇见过。

``` obj1.method.call(obj2) 将属于obj1的方法用在obj2上，因此this就会变化，使用obj2调用了obj1的方法，调用者是obj2, this就会变化成obj2
obj1.method.call(obj2) 将属于obj1的方法用在obj2上，因此this就会变化，使用obj2调用了obj1的方法，调用者是obj2, this就会变化成obj2
```

例子：

```javascript
var obj1 = {
	name : "Gtom",
	getName : function(){
		alert(this.name)
	}
}
var obj2 = {
	name:"Gjerry" 
	}
obj1.getName.call(obj2);//输出Gjerry
```

Apply与call的功能完全一样，但是导入的参数有所不同：

```javascript
fun.apply(thisArg[, argsArray])
fun.call(thisArg[, arg1[, arg2[, ...]]])
```

第一个参数都一样都是新的this object

apply的第二个参数是一个数组，而call的第二个往后的参数都是参数列表。



bind是什么呢，定义上是这么写的：

```
bind() 函数会创建一个新函数（称为绑定函数），新函数与被调函数（绑定函数的目标函数）具有相同的函数体...这种行为就像把原函数当成构造器。提供的 this 值被忽略...
```

说的有点复杂，但是好像也和this有关系，所以会和apply/call 放一起。我们用例子来看：

bind的用法与call类似

```
fun.bind(thisArg[, arg1[, arg2[, ...]]])
```
例：

```javascript
var obj = {
    x: 81,
};
var foo = {
    getX: function() {
        return this.x;
    }
}
console.log(foo.getX.bind(obj)());  //81
console.log(foo.getX.call(obj));    //81
console.log(foo.getX.apply(obj));   //81
```

三个输出的都是一样，但是注意看使用 bind() 的方法，他后面多了对**括号**。

所以可以用bind(obj ) = 一个函数 ，但是不立刻执行，bind(obj)() 之后才执行(回调执行)，但是apply(obj)/call(obj) 就会立即执行函数。

总结区别：

- apply 、 call 、bind 三者都是用来改变函数的this对象的指向的；

- apply 、 call 、bind 三者第一个参数都是this要指向的对象，也就是想指定的上下文；
- apply 、 call 、bind 三者都可以利用后续参数传参；
- bind 是返回对应函数，便于稍后调用；apply 、call 则是立即调用 。

### apply /call常用用法

1. 伪数组使用数组的方法处理

   伪数组，就是像数组一样有 `length` 属性，也有 `0`、`1`、`2`、`3` 等属性的对象，看起来就像数组一样，但不是数组。但是如果想利用数组(或者其他对象)的方法(push,pop ,Math.max...)来处理伪数组，就可以用apply

   常见的伪数组：arguments 对象;getElementsByTagName , document.childNodes返回NodeList对象等

   ```javascript
   var domNodes = Array.prototype.slice.call(document.getElementsByTagName("*"));
   ```

2. [面试题(其实和1 是一样的原理，看一下新的应用场景)](https://segmentfault.com/a/1190000000375138?page=1)

   问题1.

   ```
   让console.log传入多个参数。我传入参数的个数是不定的。
   ```

   答案

   ```javascript
   function log(){
     console.log.apply(console, arguments);
   };
   ```

   问题2 

   ```
   每一个log消息添加一个"(app)"的前辍
   ```

   答案：

   ```javascript
   function log(){
     var args = Array.prototype.slice.call(arguments);
     args.unshift('(app)');

     console.log.apply(console, args);
   };
   ```

   ### bind的常用用法

   ```javascript
   var bar = 1;
   var foo = {
       bar : 2,
       eventBind: function(){
         alert(this.bar);        //2 
           var _this = this;      
           var f = function() {
             alert(this.bar);    // 1
             alert(_this.bar);   //2   
           };
        f();
       }
   }
   foo.eventBind();
   ```

   这里的 this值在变化，特别f()里面的的this值指向了windows，因此要是想访问原来的this值，需要用_ _this_来“保留”住这个this.具体看[function Part4 闭包](https://github.com/GhostTomX/ECMAScriptNode-Notes/blob/master/js/07_Function.md)

   类似的前端场景里, var f 一般是一个event函数,或者是一个getX的保护X的外部接口

   ```javascript
   var bar = 1;
   var foo = {
       bar : 1,
       eventBind: function(){
           var _this = this;
           $('.someClass').on('click',function(event) {
               /* Act on the event */
               console.log(_this.bar);     //1
           });
       }
   }
   ```

   使用bind()会简洁很多:

   ```javascript
   var bar = 2;
   var foo = {
       bar : 1,
       eventBind: function(){
           var f = function() {
             alert(this.bar); // 1 而不是 2 !!
           }.bind(this);
        f();
       }
   }
   foo.eventBind();
   ```


将enentBind的this值(foo这个对象) 传入到这个方法的函数里面, 这样里面的每次调用f的时候就会把f的this动态改成了foo而不是原来的window

为什么这里不用apply呢?因为我想让这个f在我想调用的时候就调用.如果我改成apply那么即使我后面没有 f() ;  这个函数也会调用起来.但是如果我用bind()的话,没有f()这个函数就调用不起来.

因此,实际上bind()就是包装过的apply():

```javascript
Function.prototype.bind = function(ctx) {
    var fn = this;
    return function() {
        fn.apply(ctx, arguments);
    };
};
```

