要想解释清楚什么是闭包，得先弄清楚作用域链

什么是作用域链
---
作用域链，顾名思义就是由作用域链串起来的一条链

当代码在一个环境中执行时，会创建变量对象的一个作用域链，，它的用途，是保证对执行环境有权访问的所有变量和函数的有序访问。
总结来说，就是保证能有有序访问各个变量和函数

作用域链的前端，始终都是当前执行的代码所在环境的变量对象。如果这个环境是函数，则将其活动对象作为变量对象。活动对
象在最开始时只包含一个变量，即 arguments 对象（这个对象在全局环境中是不存在的）。作用域链中的下一个变量对象来自包含（外部）环境，而再下一个变量对象则来自下一个包含环境。这样，一直延续到全局执行环境；全局执行环境的变量对象始终都是作用域链中的最后一个对象。

是不是有点懵？

来看代码

``` JavaScript
  var color = "blue"; 
  function changeColor(){ 
     var anotherColor = "red"; 
     function swapColors(){ 
       var tempColor = anotherColor; 
       anotherColor = color; 
       color = tempColor; 

   // 这里可以访问 color、anotherColor 和 tempColor 
     } 
   // 这里可以访问 color 和 anotherColor，但不能访问 tempColor 
    swapColors(); 
  } 
  // 这里只能访问 color 
  changeColor(); 
```
上面这段代码有3个执行环境：全局环境、changeColor()的局部环境和swapColors()的局部环境，从上往下执行程序，首先是全局环境放在作用域链上的最末端，然后是changeColor局部环境放在全局环境的后面，再这个局部环境中又有一个swapColors的局部环境，于是紧跟着changeColor的后面

现在作用域链长这个样子 （最末端）全局环境→→→→→changeColor→→→→→swapColors（最前端）

changeColor局部环境可以访问全局环境中的color变量，但不能访问swapColors中的变量，swapColors局部环境可以访问changeColor中的anotherColor变量，也可以访问全局环境的变量。内部环境可以通过作用域链访问所有的外部环境，但外部环境不能访问内部环境中的任何变量和函数，局部环境会先在自己的变量环境中查找，如果找不到，再搜索上一级作用域，如果找到了，就停止查找，如果找不到，就继续向上查找，直到全局环境。

解释完作用域链，我们来看下闭包

什么是闭包
---
根据上面作用域链的定义，我们可以看出：外部环境是访问不了内部环境的，作用域链不会向下搜索

但总有些时候是需要外部去访问内部的，所以就有了闭包

闭包：是指有权访问另一个函数作用域中的变量的函数

所以，闭包也是一个函数，那创建闭包就是创建一个函数咯，常见创建闭包方法方法就是，在一个函数内部创建另一个函数

实现闭包方法
---
先看一段代码
``` JavaScript
  function f1(){
    n = 999;
    function f2(){
      console.log(n);　　
    }
  }
  
  // undefined
  
  // 函数f2包含在f1内，根据上面讲的作用域链，f2的局部环境可以访问它上一层的f1中的所有变量
  // 但f1不能访问它下一层的f2中的变量
  
```
既然f2可以访问f1中的局部变量，那只要把f2作为返回值，在f1外部就可以读取它的内部变量了
``` JavaScript
  function f1(){
    n = 999;
    function f2(){     // f2就是闭包
      console.log(n);
    }
    return f2;
  }
  var result = f1();　　//返回的是f2函数
  result();
  
  //999
```

闭包的用途
---
* 读取函数内部的变量
* 让这些变量一直保存在内存中

看一段代码
``` JavaScript
  function f1(){
    var n = 999;
    nAdd = function(){
      n += 1;
    }
    function f2(){
      console.log(n);
    }
    return f2;
  }
  var result = f1();
  result();　　
  nAdd();　　
  result();
  
  // 999
  // 1000
```
在这段代码中，result实际上就是闭包f2函数。它一共运行了两次，第一次的值是999，第二次的值是1000。这证明了，函数f1中的局部变量n 一直保存在内存中，并没有在f1调用后被自动清除。

为什么会这样呢？原因就在于f1是f2的父函数，而f2被赋给了一个全局变量，这导致f2始终在内存中，不会再调用结束后，被垃圾回收机制回收。

这段代码中另一个值得注意的地方，就是‘nAdd=function(){n+=1}’这一行，首先在nAdd前面没有使用var关键字，因此nAdd是一个全局变量，而不是局部变量。其次，nAdd的值是一个匿名函数，而这个匿名函数本身也是一个闭包，所以nAdd相当于是一个setter，可以在函数外部对函数内部的局部变量进行操作。

注意
---
闭包虽好用，但却要谨慎使用，因为闭包会使得在函数中的变量都保存在内存中，内存消耗大，导致网页性能问题，在IE中，甚至会导致内存泄漏

解决方法
``` JavaScript
  // 创建函数
  var compareName = createComparosonFunction('name')

  //调用函数
  var result = compareName({ name: '123' }, { name: '123' })

  // 解除对匿名函数的引用   (以便释放内存) 
  compareName = null
```
在退出函数之前，设置函数为null，是为了解除对函数的引用，等于通知垃圾回收例程将其清除。随着匿名函数的作用域链被销毁，其他作用域
（除了全局作用域）也都可以安全地销毁了

参照博客
---

https://www.cnblogs.com/duanlianjiang/p/5036671.html

红宝书《JavaScript高级程序设计》


