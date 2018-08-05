---
title: Javascript 对象简介
date: 2014/1/12 22:03:29
categories:
- Shit Done
tags:
- Javascript
---
#### javascript对象的属性检索

今天提提javascript中对象值得一提的东西。
##### 1. 如果访问一个对象的属性不存在，那么返回undefined。

``` javascript
// if a property is not exsist ,then return undefine
var stooge = {
    firstName: "Jerome",
    'last_name': "Howard" 
};

console.log(stooge.firstName);  //Jerome
console.log(stooge.nikeName); // undefine
```
##### 2. 我们可以给用 "||" 运算符给对象的属性赋默认值。

``` javascript
//use '||' operator to set varliable default value;
var nikeName1  = stooge.nikeName || "none";
if(nikeName1){
    console.log("nikeName1 is: " + true); //nikeName1 is true
}
console.log(nikeName1);  //none
```

<!--more-->
##### 3. 如果我们访问一个对象的不存在的属性的属性，那么将会跑TypeError错误。

``` javascript
 console.log(stooge.nikeName.length); //TypeError
```

为了避免这种情况的发生，我们可以使用"&&"运算符.

``` javascript
//to prevent this case, we can use '&&' operator.
var nikeNam2 = (stooge.nikeName && stooge.nikeName.length)
console.log(nikeNam2);
if(nikeNam2){
    console.log("nikeNam2 is: " + true);
}else{
    console.log("nikeNam2 is: " + false);
}
```
##### 4. 我们都知道javascrip是基于原型继承实现的面向对象，与java基于类的继承不同的一种面向对象实现方式。javascript中每个新创建的对象都是继承了另一个对象的属性。为了让这种关系明显的显示出来我们可以定义如下的方法。

``` javascript
//Prototype 原型
console.log(Object.beget); //undefine
if (typeof Object.beget !== 'function') {
    Object.create = function(o){
        var F = new Function();
        F.prototype = o;
        return new F();
    };
}
var another_stooge = Object.create(stooge); //another_stooge 继承于stooge。
console.log("another_stooge: " + another_stooge.firstName); //Jerome（继承来的属性）
console.log("another_stooge: " + another_stooge.fun()); //fun (继承来的方法)
```

关于javascript的原型继承，属于javascript中很大的一部分内容，我们后面还会讲到。
#### javascript对象属性的遍历
##### 5. 既然javascript中的对象继承了另一个对象的属性和方法，那么我们在访问这个对象的时候，通常我们会有这样的需求，就是想访问这个对象本身扩展的所有的属性。解决方法如下。

``` javascript
//我们便于测试先给another_stooge添加自己的属性和方法。
another_stooge.nikeName = "Bob"; 
another_stooge.fun2 = function(){
    console.log("another_stooge's function....");
}

//过滤继承而来的属性和方法
for(name in another_stooge){
    if(another_stooge.hasOwnProperty(name)){
        console.log(name);  //nikeName fun2
    }
}
```
##### 6.如果我们有让对象的属性按我们想要的顺序输出，那么我们可以这样做。

``` javascript
//定义一个循环变量
var i;
//定义顺序数组
var properties = [
    'nikeName',
    'firstName',
    'last_name',
    'fun',
    'fun2'
];
//循环按顺序输出属性。
for(i = 0; i < properties.length; i++){
    console.log(properties[i] + " : "+ another_stooge[properties[i]]);
}
```
#### javascript对象属性的删除
##### 6.当对象的属性与继承而来的属性同名时，会覆盖继承而来的属性值。但删除操作对于对象自己的属性是有作用的对继承而来属性是没有作用的。

``` javascript
//delete
console.log(another_stooge.firstName) // Jerome
another_stooge.firstName = "Micheal";//覆盖继承而来的属性。
console.log(another_stooge.firstName) //Micheal

delete another_stooge.firstName;   // true
console.log(another_stooge.firstName) // Jerome //删除后，又访问了继承而来的值，

delete another_stooge.firstName; //true; 但是没有真实的删除。
console.log(another_stooge.firstName) // Jerome
```

这是因为javascript基于原型的设计，如果可以删除继承而来的属性，那么将会影响所有继承于这个对象的其他对象，这样整个javascript的对象系统就乱了。

BTW, 祝大伙平安夜快乐，圣诞快乐 ：)
