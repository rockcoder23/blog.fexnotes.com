---
title: Javascript 中的正则
date: 2014/8/15 17:47:30
categories:
- Shit Done
tags:
- Javascript 
- 正则表达式
---

## 一、RegExp对象

构造正则表达式：

``` javascript
//方式1：
var re = /\w+/;   //最常用的方式

//方式2：
var re = new RegExp("\\w+");  //注意转义
```
### 1. reg.test(str)

描述：test() 方法执行一个检索，用来查看正则表达式与指定的字符串是否匹配。返回 true 或 false。

``` javascript
function testinput(re, str){
    var midstring;
    if (re.test(str)) {
        midstring = " contains ";
    } else {
        midstring = " does not contain ";
    }
    console.log(str + midstring + re.source);
}

testinput(/c{2}/, 'cainiao'); 
//"cainiao does not contain c{2}"
```
- reference: [MDN参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/RegExp/test)

<!-- more -->
### 2. reg.exec(str)方法
- exec() 方法为指定的一段字符串执行搜索匹配操作。它的返回值是一个数组或者 null。

``` javascript
var re = /d(b+)(d)/ig;
var result = re.exec("cdbBdbsbz");
console.log(result); 
//["dbBd", "bB", "d", index: 1, input: "cdbBdbsbz"] 
```
- 返回的数据：

<table class="fullwidth-table">
 <tbody>
  <tr>
   <td class="header">对象</td>
   <td class="header">属性/索引</td>
   <td class="header">描述</td>
   <td class="header">例子</td>
  </tr>
  <tr>
   <td rowspan="4"><code>result</code></td>
   <td><code>[0]</code></td>
   <td>正则表达式最后的匹配项</td>
   <td><code>dbBd</code></td>
  </tr>
  <tr>
   <td><code>[1], ...[
    <i>
     n</i>
    ]</code></td>
   <td>子表达式匹配项</td>
   <td><code>[1] = bB<br>
    [2] = d</code></td>
  </tr>
  <tr>
   <td><code>index</code></td>
   <td>第一个匹配项在原字符串中的索引</td>
   <td><code>1</code></td>
  </tr>
  <tr>
   <td><code>input</code></td>
   <td>方法输入的参数字符串</td>
   <td><code>cdbBdbsbz</code></td>
  </tr>
  <tr>
   <td rowspan="5"><code>re</code></td>
   <td><code>lastIndex</code></td>
   <td>下一次执行匹配开始索引的位置.</td>
   <td><code>5</code></td>
  </tr>
  <tr>
   <td><code>ignoreCase</code></td>
   <td>指"<code>i</code>" 标识是否启用</td>
   <td><code>true</code></td>
  </tr>
  <tr>
   <td><code>global</code></td>
   <td>指"<code>g</code>" 标识是否启用</td>
   <td><code>true</code></td>
  </tr>
  <tr>
   <td><code>multiline</code></td>
   <td>指"<code>m</code>" 标识是否启用</td>
   <td><code>false</code></td>
  </tr>
  <tr>
   <td><code>source</code></td>
   <td>正则表达式的文本表示</td>
   <td><code>d(b+)(d)</code></td>
  </tr>
 </tbody>
</table>
- reference：[MDN参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/RegExp/exec)
## 二、String对象
### 1. str.search(reg) vs reg.test(str)

*描述：search() 方法执行一个查找，看该字符串对象与一个正则表达式是否匹配。参数reg为一个正则表达式对象，否则隐式调用new RegExp(reg)进行转换。
如果匹配成功，则 search 返回正则表达式在字符串中首次匹配项的索引。否则，返回 -1。

``` javascript
function testinput(re, str){
  var midstring;
  if (str.search(re) != -1){
    midstring = " contains ";
  } else {
    midstring = " does not contain ";
  }
  console.log (str + midstring + re);
}
testinput(/db*/, 'sdbbdee'); //sdbbdee contains /db*/ 
```

＊reference: [MDN参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/search)
### 2. str.match(reg)  vs  reg.exec(str)
- 描述：当字符串匹配到正则表达式（regular expression）时，match() 方法会提取匹配项。reg参数为一个正则对象，若不是，则隐式地调用new RegExp(reg) 进行转换。返回数组，不匹配返回null。

``` javascript
var str = "For more information, see Chapter 3.4.5.1";
var re = /(chapter \d+(\.\d)*)/i;  //不是用g的情况
var found = str.match(re);
console.log(found);
//["Chapter 3.4.5.1", "Chapter 3.4.5.1", ".1", index: 26, input: "For more information, see Chapter 3.4.5.1"] 
```

``` javascript
var str = "ABCDEFGHIJKLMNOPQRSTUVWXYZabcdefghijklmnopqrstuvwxyz";
var regexp = /[A-E]/gi;  //使用g时，found不存在index和input属性
var found = str.match(regexp);
//['A', 'B', 'C', 'D', 'E', 'a', 'b', 'c', 'd', 'e']
```

*reference：[MDN参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/match)
### 3. str.replace(regexp|substr, newSubStr|function[,  flags])
- 描述：replace() 方法使用一个替换值（replacement）替换掉一个匹配模式（pattern）在原字符串中某些或所有的匹配项，并返回替换后的字符串。这个替换模式可以是字符串或者正则表达式，替换值可以是一个字符串或者一个函数。
- 参数
  regexp：
  一个 RegExp 对象。该正则所匹配的内容会被第二个参数的返回值替换掉。
  substr：
  一个要被 newSubStr 替换的字符串。
  newSubStr：
  替换掉第一个参数在原字符串中的匹配部分。该字符串中可以内插一些特殊的变量名。
  function：
  一个用来创建新子字符串的函数，该函数的返回值将替换掉第一个参数匹配到的结果。该函数的参数描述请参考 Specifying a function as a parameter 小节。.
  flags：
  注意：flags 参数在 v8 内核（Chrome and NodeJs）中不起作用。一个字符串，用来指定 regular expression flags 或其组合。在 String.replace method 中使用 flags 参数不是符合标准。使用一个带有相应标志（flags）的正则表达式（RegExp）对象来代替此参数。该参数的值应该是下面的一个或多个字符，具体作用见下：
  g   全局替换
  i   忽略大小写
  m   多行模式
- 返回：
   一个新字符串，其中匹配模式的某些或所有匹配项被替换为替换值。该方法并不改变调用它的字符串本身,而只是返回替换后的字符串.

``` javascript
var re = /apples/gi;
var str = "Apples are round, and apples are juicy.";
var newstr = str.replace(re, "oranges");
print(newstr);
//"oranges are round, and oranges are juicy."
```

等同于：

``` javascript
var str = "Apples are round, and apples are juicy.";
var newstr = str.replace("apples", "oranges", "gi");
print(newstr);
//"oranges are round, and oranges are juicy."
```

``` javascript
var re = /(\w+)\s(\w+)/;
var str = "John Smith";
var newstr = str.replace(re, "$2, $1");
print(newstr); //'Smith John'
```
- reference: [MDN参考](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/replace#Specifying_a_string_as_a_parameter)
### 4. str.split(seperator, limit)
- 描述：用来分割字符串的字符（串）。separator 可以是一个字符串或正则表达式。 如果忽略 separator，则返回的数组包含一个由原字符串组成的元素。如果 separator 是一个空字符串，则 str 将会转换成一个由原字符串中字符组成的数组。 limit一个整数，限定返回的分割片段数量。split 方法仍然分割每一个匹配的 separator，但是返回的数组只会截取最多 limit 个元素。

＊注： 如果 separator 是一个正则表达式，且包含捕获括号（capturing parentheses），则每次匹配到 separator 时，捕获括号匹配的结果将会插入到返回的数组中。然而，不是所有浏览器都支持该特性。　

``` javascript
var myString = "Hello 1 word. Sentence number 2.";
var splits = myString.split(/(\d)/);
console.log(splits);
//["Hello ", "1", " word. Sentence number ", "2", "."]
```
- reference: [MDN参考](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String/split)
