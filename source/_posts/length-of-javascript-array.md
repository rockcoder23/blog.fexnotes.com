---
title: Javascript 数组诡异的 length 属性
date: 2014/4/28 20:03:29
categories:
- Shit Done
tags:
- Javascript
---

#### 一、 javascript数组简介
-   javascript中数组其实本质是一个对象,只是javascript作者想用一个对象来模拟数组这种数据结构的特性。我们来看下面这段代码：

``` javascript
var empty = [];
var numbers = ['zero', 'one', 'two', 'three', 'four'];
var numbers_object = {
    0: 'zero',
    1: 'one',
    2: 'two',
    3: 'three',
    4: 'four'
}
console.log(empty[0]) //undefine  访问不存在的元素返回undefine
console.log(numbers[1]) //one
```

其实numbers与numbers_object本质是一样都是对象，但是为了让numbers比一般的对象更具数组特性，让所有想具有数组特性的对象，如：numbers，继承自Array.prototype，而numbers_object继承自Object.prototype,这样numbers具有了大量了一般对象没有的有用的方法和我们下面将要讨论的length属性。

<!--more-->
-  javascript数组对象的可以是混合类型的

``` javascript
var misc=[
    'string', 98.6, true, false, null, undefine, ['nested', 'array'],
    {name: 'Bob'}, NaN
]
```
#### 二、javascript诡异的length属性

先来看一段代码，可以对照后面的注释答案，看看与你想的是否一样。

``` javascript
var myArray = [];
console.log(myArray.length); // 0

//有一个元素，但是length却为0。
myArray[4294967300] = 'big number';
console.log(myArray.length); // 0 

myArray[0] = 'zero';
myArray[1] = 'one';
console.log(myArray.length); // 2

//默认转化为数字2
myArray['2'] = 'two' ;
console.log(myArray.length); // 3


//一般的字符串属性不影响length值。
myArray['prop1'] = 'prop1';
console.log(myArray.length); //3

//直接设置一个100，数组对象的元素个数为4个，但是length却是101
myArray[100] = '100';
console.log(myArray.length); // 101

//同理
myArray['200'] = '200';
console.log(myArray.length); // 201

myArray[4294967294] = 'big number';
console.log(myArray.length); // 4294967295

//length的最大值为：4294967295
myArray[4294967300] = 'another big number';
console.log(myArray.length); // 4294967295


myArray.length = 0;
console.log(myArray);     //[];  //数组被清空。

//不影响非数字属性。
console.log(myArray['prop1']) ;   //prop1
```

估计这会让你大跌眼镜，这个length好诡异把。
好了我们来总结下。
#### 三、javascript数组length总结

产生上面的困惑与[]运算符的二义性和length的特性有关。
##### 1. [] 运算符的二义性：同时为数组下标运算符和对象属性存取符。
-  []运算符后面接的元算元可以是变量，直接量，表达式。而'.'运算符右边的运算元必须是标识符,所以"abc.def" , '1' , '.'这些属性名只能使用[]运算符访问。所以可以理解为[]功能比'.'运算符更强大，而'.'运算符比[]运算符的可读性高。
-  []运算符会把它包含的表达式转换成一个字符串，如果该表达式有tostring方法就使用该方法转换。并且这个字符串会被用于做属性名，当但是当对象同时也是数组，且这个字符串能被转换为下标数字时，如：myArray['2'] ,这个属性的存在却影响了数组的length的值。而myArray['prop']却不会影响length值。
##### 2. length的特性
-   如果这个[]运算符中的字符串看起来像大于等于当前的length并且小于429496709正整数，那么length就被重新赋值为新的下标加1. 根据ECMAScript262标准，数下标必须大于等于0 小于2 e32 - 1。
-   你可以直接给length指定一个值，如果length值大于当前的数组对象内元素个数，数组对象不会得到更多的空间，但是把length设置为小于当前数组对象内元素个数那么大于length的属性将会直接删除，可见length是很诡异，可以说这是javascript的一个失败的设计，应当慎用。但不会影响到对象的非数字属性。如： myArray['prop'];

对照总结再把上面的代码对照一遍，你就明白了为什么会是这个结果了。
#### 四、javascript数组元素的删除

我们怎么正确的删除一个数组的元素呢？
1. 错误的删除方式：

``` javascript
//先声明一个数组
var myArray2 = [1, 2, 3, 4, 5, 6, 7, 8, 9, 10];
//尝试删除元素
delete  myArray2[2]; 
console.log(myArray2); //[1, 2, undefined × 1, 4, 5, 6, 7, 8, 9, 10] 
console.log(myArray2.length);  //10
```

可见使用delete删除一个元素 虽然删除了但是还占着一个坑，后面的元素索引没有减一。长度还是10
究其原因是delete把myArray2当作对象，删除了其一个属性而已。
1. 正确了删除方式：

``` javascript
//从第下标为2的元素开始删除，删除一个元素。
myArray2.splice(2, 1);  
console.log(myArray2);  //[1, 2, 4, 5, 6, 7, 8, 9, 10]
console.log(myArray2.length);  //9  
```

调用splice方法，最后长度为9。达到了删除数组元素的目的。
