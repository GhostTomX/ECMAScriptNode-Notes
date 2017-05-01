

# 函数中的this

### 参考资料

-    [JavaScript高级程序设计（第3版 )Chp 05&07](https://book.douban.com/subject/10546125/)

-    [Javascript的this用法](http://www.ruanyifeng.com/blog/2010/04/using_this_keyword_in_javascript.html)

-    [ JavaScript设计模式与开发实践 Chp 02](https://book.douban.com/subject/26382780/)

        实用的两句话：


1. this的指向在函数定义的时候是确定不了的，只有函数执行的时候才能确定this到底指向谁

2. this指向的是最接近它的调用它的对象

   this的调用方法：

   1. 对象的方法中的this : 由于方法总是对象调用的，所以指向调用方法的这个对象

      ```javascript
      var obj = {
        name:"Gtom",
        getA: function() {
          alert(this.name);
        }
      }
      obj.getA();   // 输出Gtom
      ```

   2. 普通函数中，指向全局对象（普通函数的调用者，windows）

      ```javascript
      name = "Gtom";
      var getName = function(){
        alert(this.name);
      }

      getName();//输出Gtom
      ```

      **注意**：与1类似的2;调用的是一个变量。变量指向了一个对象的方法。但是调用者还是windows而不是对象。

      ```javascript
      var obj = {
        name:"Gtom",
        getName: function() {
          alert(this.name);
        }
      }
      name = "Gjerry";
      var getName2 = obj.getName;
      getName2(); // 输出Gjerry
      ```

   3. 构造函数（调用需要用new）中： 指向返回的对象

      ```javascript
      var Myclass=function(){
        this.getName = function(){
        alert(this.name);
        }
      }
      name = "Gjerry;"
      var obj = new Myclass();
      obj.name = "Gtom";//效果与在构造函数里写this.name = "Gtom";一样
      obj.getName(); //输出Gtom
      ```

      **注意**：Myclass 返回了一个对象的时候，变量会指向那个对象而不是这个函数本身的对象，此时的this就会指向调用者，就是返回的这个对象。

      ```javascript
      var Myclass=function(){
        this.name = "Gtom";
        this.getName = function(){
        alert(this.name);
        }
        return {
      	name : "Gdavid"
        }
      }
      name = "Gjerry;"
      var obj = new Myclass();
      alert(obj.name)//输出Gdavid
      ```

   4. Function.prototype.call/apply 调用： 动态改变传入的函数的this.

      之前在了解[继承](https://github.com/GhostTomX/ECMAScriptNode-Notes/blob/master/js/06_Object.md)的时候使用过call/apply

      obj1.method.call(obj2) 将属于obj1的方法用在obj2上，因此this就会变化，使用obj2调用了obj1的方法，调用者是obj2, this就会变化成obj2

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

      ​

   ​

   ​

