---
title: Javascript 中的 this 对象
date: 2014/1/12 22:03:29
categories:
- Shit Done
tags:
- JVM
- 内存模型
---

最近看道大师的 javascript：The Good part一书中谈道在javascript中函数的调用的四种方式，今天就来说说这四种不同方式下的this变量的含义。以下代码建议在chrome下运行。
#### 1. 函数的方法调用模式

当function作为对象的属性调用时，我们称之为方法。看以下代码结果。

``` js
var myObject = {
    value: 0,
    increment: function(inc){
        // console.log(typeof inc);  //返回表示数据类型的字符串。
        // console.log(typeof inc === 'number') //inc 如果是number类型，则输出true；
        console.log(this);//谁调用this 指向谁。输出myObject对象
        this.value += typeof inc === 'number' ? inc : 1;
    }
};
myObject.increment();
console.log(myObject.value); //1
myObject.increment(2);
console.log(myObject.value); //3
myObject.increment('0');
console.log(myObject.value); //4
```

可见，在函数作为对象的属性时，被调用时this指向这个对象。
#### 2. 函数的函数调用模式

当函数并非一个对象的属性时，那它就当做一个函数来调用。

``` js
myObject.double = function(){
    console.log(this); //myObject
    console.log("outter function's this: " + this); //myObject
    var helper = function(){
        //绑定到了全局的对象window；
        console.log("inner function's this: " + this); //window
    }
    helper();
};
myObject.double();
```

我们为myObject增加一个double方法，并且在double方法中定义了一个helper函数，并调用它。我们可以可以看到这个时候，第一个输出的this指向myObject，原理跟第一个一样。然而，helper函数里面的this却指向了window全局对象。正如道大师所说这是javascript设计中的一个错误。为了能让内嵌的函数能访问包裹它的函数的上下文，我们可以如下设计。

//利用js函数闭包可以访问上一层的函数的上下文的特性。
//把对象本身赋值给变量that。这样就实现了对外层函数的操作。

``` js
myObject.double = function(){
    var  that = this;
    var helper = function(){
        console.log(that.value);
        that.value = that.value + 1;
        console.log(that.value);
    }
    helper();
};
myObject.double();
```

我们习惯上把包裹函数的this对象赋值给that，利用javascript闭包的特性，就可以使用内嵌函数访问包裹函数的上下文。这个特性经常会用到。
#### 3. 函数的构造器调用模式

如果一个函数的创建是为了让new调用，那么这个函数成为构造器，当用new调用时，会创建一个新对象，并且这个对默认链接到这个构造器的prototype属性的对象上。见如下代码：

``` js
var Quo = function(string){
    console.log("inner constructor this: ");
    console.log(this);
    this.status = string;
    console.log(this);
}

//给所有从Quo构造的对象添加get_status方法。
Quo.prototype.get_status = function(){
    console.log("inner get_status this: ");
    console.log(this);
    return this.status;
};
var myQuo = new Quo("confused..");
console.log("myQuo.get_status: " + myQuo.get_status());
```

这个时候构造器内的this指向了新创建的对象。
#### 4. 函数的apply调用模式

apply方法让我们构建一个参数数组传递给调用函数，它也允许我们选择this的值，apply方法接收两个参数，第一个是要绑定给this的值，第二个就是一个参数数组；

``` js
var status_Object = {
    status: 'A-OK'
};

//status_Object 并没有继承自Quo.prototype，但我们可以在status_Object
//调用get_status方法。
var status = Quo.prototype.get_status.apply(status_Object);
console.log(status);
```

结合第三种调用方式中的代码，可以看出，因为get_status方法的参数为空，所以我们只传入了status_Object，从输出（get_status方法内的输出）可见this指向了status_Object。但当apply()的参数为空时，默认调用全局对象。
