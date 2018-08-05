---
title: (译)ES6 Arrow Functions in Depth
date: 2015/10/25 00:03:29
categories:
- Shit Done
tags:
- ES6
- array function
---
> [原文地址](https://ponyfoo.com/articles/es6-arrow-functions-in-depth)

继续每日更新[es6-in-depth](https://ponyfoo.com/articles/es6-arrow-functions-in-depth)文章，今天我们将会讨论箭头函数。 在前一篇文章中我们涉及到了[解构](https://ponyfoo.com/articles/es6-destructuring-in-depth)和[模板字符串](https://ponyfoo.com/articles/es6-template-strings-in-depth)。我尽力去覆盖所有ES6的特性，甚至我们会讨论到ES7.我发现总结这些特性同时也很容易让我印象更加深刻。

因为你已经在阅读这一系列的文章，我建议你[安装Babel和babel-node]((https://ponyfoo.com/articles/universal-react-babel#setting-up-babel)，并把例子复制到文件里面，然后你可以通过`babel-node yourfile`在控制台中运行。自己在控制台中运行这些例子或者对它们稍作一些修改将对你更好的理解这些特性很有帮助。即使你只是添加`console.log` 语句来弄懂其中的缘由。

> 让我们开始今天的话题吧

在之前的文章中我们一起见识过了箭头函数，然后我们没有更多的对她进行解释。这篇文章我们将主要关注箭头函数并且先抛开其他的ES6特性。我认为这是最好的总结ES6的方式--把每个特性分篇，并且在每篇中逐渐的增加一些其他相关联的概念，这样有助于我们理解他们是怎么协作的。我已经关注到在ES6中有很多非常棒的相互协助的例子。慢慢的深入ES6中的语法和特性仍然是非常重要的而不是一头扎进温水中，因为你将无法适应之后的水温---这可能是一个失败的比喻。我们继续。
### 箭头函数

箭头函数是很多现代的语言都提供的一个功能，也是我从C# 转到Javascript后很想念的特性之一。非常幸运，她现在是ES6中一部分，我们能直接在javascript中使用。箭头函数的语法也是十分的形象。我们已经有了匿名函数，但是有一个精简的替代品也是不错的选择。

如果我们有一个单独的参数和只是想返回一个表达式的结果，下面就是使用箭头函数的写法。

``` js
    [1, 2, 3].map(num => num * 2)  //<- [2, 4, 6]
```

等同下面ES5的写法。

``` js
    [1, 2, 3].map(function (num) {
        return num * 2  //<- [2, 4, 6]
    })
```

如果我们想声明更多参数或者不需要参数，我们得使用圆括号。

``` js
    [1, 2, 3, 4].map((num, index) => num * 2 + index) //<- [2, 5, 8, 11]
```

你可能还有其他的语句，而不只是返回一个表达式。这种情况你将使用大括号。

``` js
    [1, 2, 3, 4].map(num => {
        var multiplier = 2 + num
        return num * multiplier
    })
    //<- [3, 8, 15, 24]
```

你也可以使用圆括号添加更多的参数。

``` js
    [1, 2, 3, 4].map((num, index) => {
        var multiplier = 2 + index
        return num * multiplier
    })
    //<- [2, 6,12, 20]
```

尽管如此，有时候你最好还是使用命名函数声明，理由如下：
-  `（num, index）=>`只是比`function (num, index)`稍微简短了点。
-  `function` 关键字能够让你为函数取名，提高代码质量。
-  当一个函数有多个参数和多条语句时，我不得不说不太可能因为多了六个字符就造成什么影响(注:这个地方没弄明白作者意图，为啥是六个字符？)
-  然而，命名函数能够增加上下文那才使得加了六个字符(function)变得非常有价值

如果我们想返回一个对象常量，我们得使用圆括号包裹下。这样对象常量才不会被解释为语句声明块(这也会导致静态的语法错误，因为`number: n`在下面的例子中不是一个合法的表达式)。第一个例子中解释`number`作为属性名，n作为表达式。因为我们这是在一个代码块里面，并且没有返回任何东西，map遍历值返回`undefined`。第二个例子，在number和n之后 `something:else`将无法被编译，所以抛出`SyntaxError`异常。

``` js
[1, 2, 3].map(n => { number: n })
// [undefined, undefined, undefined]
[1, 2, 3].map(n => { number: n, something: 'else' })
// <- SyntaxError
[1, 2, 3].map(n => ({ number: n }))
// <- [{ number: 1 }, { number: 2 }, { number: 3 }]
[1, 2, 3].map(n => ({ number: n, something: 'else' }))
/* <- [
  { number: 1, something: 'else' },
  { number: 2, something: 'else' },
  { number: 3, something: 'else' }]
*/
```

箭头函数有个一个相当好的特点是他们已经预先绑定了词法词法作用域。这表明你将可以为了保持深度嵌套的函数上下文的`var self = this` 和`.bind（this）`类似的hack说拜拜了。

``` js
function Timer () {
  this.seconds = 0
  setInterval(() => this.seconds++, 1000)
}
var timer = new Timer()
setTimeout(() => console.log(timer.seconds), 3100)
// <- 3
```

记住箭头函数所保持的`this`引用一位置你使用`.call` 和`.apply` 将不会改变这个上下文。这是一个看起很像bug的特性。
### 总结

使用箭头函数来定义一个自绑定上下文的匿名的函数时是很精简的，它可以极大的简化你 代码。

除非你的函数参数和语句是十分的简单易懂，否则没有理由把所有的函数都声明成箭头函数。我十分倡导使用命名函数，因为它不用注释就可以提高代码的可读性，也意味着我将每次纠结是否使用箭头函数。

最后不得不说，我认为箭头函数在函数式变成场景下是十分有用的，例如当你对列表使用`.map`, `.filter`, `.reduce`时。类似的箭头函数在异步的场景下也非常有用，因为典型的是这种场景有一大堆的回调来处理参数，这是的箭头函数可以发挥它的作用。
