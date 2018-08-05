---
title: (译)ES6 JavaScript Destructuring in Depth 
date: 2015/11/8 19:21:25
categories:
- Shit Done
tags:
- ES6
- Destructuring
---

> [原文地址](https://ponyfoo.com/articles/es6-destructuring-in-depth#special-case-import-statements)

这篇文章将专注于讨论ES6中的语言特性。以ES6中的最为有用，同时也存在一些陷阱的解构开始，我们之后将开一个系列文章来讨论ES6。
#### 简短的声明

当在不确定和有选择的情况下，你可能还是应该使用ES5和老的语法而不是因为你知道ES6就是用ES6。然而我并不是说是用ES6的语法是一种错误的方法--恰恰相反，你正看到我写关于ES6的文章。我提倡的是基于这样一个事实，确定她一定能提高我们代码的质量的时候才使用ES6, 而仅仅是因为她时一个`酷玩意儿`--虽然她是。

到目前为止，我所说的观点是用ES5来写东西，加上一些ES6的语法糖在上面将确切的改善我的代码。我推测在不久后我能快速的意识到某些场景下使用ES6是优于ES5的，但是在目前紧跟着学习她是一个不错的主意，而不要走得太极端。谨慎分析什么是最适合你的代码，并且心里记着可以有ES6这个新的选择就行。

> 这样，你将懂得怎样将新的特性学以致用，而不只是学学语法。

现在就来看看ES6中的新特性把！
#### 解构

这是ES6中最简单，也是我使用的最多的一个特性。她可以绑定属性到你需要的任意多的变量上面，并且她支持数组和对象。

``` js
    var foo = { bar: 'pony', baz: 3}
    var { bar, baz } = foo
    console.log(bar)  //<- 'pony'
    console.log(baz)  //<- 'baz'
```

 她可以十分高效的将指定属性从一个对象中解构出来。你也可以使用映射属性到一个别名。

``` js
    var foo = {bar: 'pony', baz: 3}
    var {bar: a, baz: b} = foo 
    console.log(a) //<- 'pony'
    console.log(b)  //<- '3'
```

你也可以把属性放到任意深的位置，并且完成别名的绑定。

``` js
    var foo = { bar: {deep: 'pony', dangerouslySetInnerHTML: 'lol'}}
    var { bar: {deep, dangerouslySetInnerHTML: sure} } = foo
    console.log(deep)  //<- 'pony'
    console.log(sure)  //<- 'lol'
```

默认的，没有找到的对应的属性的值将使`undefined`,  就像你通过`.`或者`[]`访问一个对象中不存在的属性一样。

``` js
    var { foo } = {bar: 'baz'}
    console.log(foo)  //<- undefined
```

如果尝试去访问对象一个不存在的深嵌套的属性，将会得到一个异常。

``` js
    var {foo: {bar}} = {baz: 'ouch'} //<- Exception  Cannot read property 'bar' of undefined
```

如果你知道她只是在ES5上进行了一层语法的包装，你将好理解很多

``` js
    var _tmp = { baz: 'ouch' }
    var bar = _temp.foo.bar //<- Exception
```

解构有个非常酷的特性她允许你不用通过中间变量来交换变量之间的值。

``` js
    function es5() {
        var left = 10
        var right = 20
        var aux
        if (right > left) {
            aux = right
            right = left
            left = aux
        }
    }

    function es6() {
        var left = 10
        var right = 20
        var aux
        if (right > left ) {
            [ right, left ] = [ left, right ]
        }
    }
```

另外一个方便的部分是解构可以让`keys`[使用表达式](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Object_initializer#Computed_property_names)。

``` js
    var key = 'such_dynamic'
    var [ [key]: foo ] = { such_dynamic: 'bar' }
    console.log(foo) //<- 'bar'
```

在ES5中，你要实现的话需要而外的语句和变量声明

``` js
    var key = 'such_dynamic'
    var baz = { such_dynamic: 'bar'}
    var foo = baz[key]
    console.log(foo)
```

你可以为变量指定默认值，当解构出来的属性值为`undefined`的时候，比较有用。

``` js
    var { foo = 3 } = { foo : 2 }
    console.log(foo) //<- 2

    var { foo = 3 } = { foo: undefined }
    console.log(foo) //<- 3

    var { foo = 3 } = { bar: 2 }
    console.log(foo) //<- 3
```

解构同样对数组有效，且看我用中括号在声明语句中解构数组

``` js
    var [a] = [10]
    console.log(a) //<- 10
```

并且我们同样可以使用相同的规则指定默认值

``` js
    var [a] = []
    console.log(a)  //<- undefined
    var [b = 10] = [undefined]
    console.log(b) //<- 10

    var [c = 10] = []
    console.log(c) //<- 10
```

当是数组的时候你还可以方便的忽略某些你不关心的元素

``` js
    var [,,a, b] = [1, 2, 3, 4, 5]
    console.log(a) //<- 3
    console.log(b)  //<- 4
```

同样你可以在函数的参数列表中使用解构

``` js
    function greet ({age, name:greeting='she'}) {
        consol.log(`${greeting} is ${age} years old.`)
    }
    greet({name: 'nico', age: 27}) //<- 'nico is 27 years old'

    greet({age: 24}) //<- 'she is 24 years old'
```

以上是你可以怎样使用解构的一个粗略的介绍，那解构一般用于什么场景呢？
#### 解构的使用场景

解构用各种使用场景，这个是最常见的一个，当你有个返回对象的方法时，解构可以通过十分简便的方法提取返回值

``` js
    function getCoords () {
        return {
             x: 10, 
             y: 22
         }
    }
    var {x, y} = getCoords()
    console.log(x) //<- 10
    console.log(y) //<- 20
```

另一个相似的但是相反的是使用结构为函数的一大串参数定义默认值，这个十分有趣，她实现了在其他语言如`Python` 和`C#`中的命名参数。

``` js
    function random ({ min = 1, max = 30 }) {
        return Math.floor(Math.random() * (max - min)) + min
    }
    console.log(random({}))  //<- 174
    console.log(random({max:24})) //<- 18
```

如果你想让形参完全是可选的，你可以改变写法如下

``` js
    function random({min = 1, max =300 } = {}) {
        return Math.floor(Math.random() * (max - min)) + min 
    }
    console.log(random()) //<- 133
```

解构还非常适用在正则表达式中，当你想给一些缺乏索引的值指定变量的时候，这里有一个解析URL的随机正则的例子，我在[StackOverflow](http://stackoverflow.com/questions/27745/getting-parts-of-a-url-regex/27755#27755)中发现的。

``` js
    function getUrlParts (url) {
      var magic = /^(https?):\/\/(ponyfoo\.com)(\/articles\/([a-z0-9-]+))$/
      return magic.exec(url)
    }
    var parts = getUrlParts('http://ponyfoo.com/articles/es6-destructuring-in-depth')
    var [,protocol,host,pathname,slug] = parts
    console.log(protocol)
    // <- 'http'
    console.log(host)
    // <- 'ponyfoo.com'
    console.log(pathname)
    // <- '/articles/es6-destructuring-in-depth'
    console.log(slug)
    // <- 'es6-destructuring-in-depth'
```

特殊的情景：`import`语句
尽管`import`语句不遵从解构的规则，但是表现有点类似。这可能是我使用的比较多的跟结构类似的场景, 尽管不是真正的解构。当你使用`import`你可以从一个模块暴露的API中获取任何你需要的值。一个使用[`contra`](https://github.com/bevacqua/contra)的例子：

``` js
    import {series, concurrent, map } from 'contra'
    series(tasks, done)
    concurrent(tasks, done)
    map(items, mapper, done)
```

注意，尽管，`import`语句有不同的语法，当对比解构，下列`import`声明没有一个是有效的。
- 使用默认的值类似于 `import { series = noop } from 'contra'` 
- 深度解构雷类似于`import {map: { series }} from 'contra'`
- 别名语法`import {map: mapAsync} from 'contra'`

以上限制的主要的原因是`import`声明自带绑定的，不是引用或者值，这是非常重要的区别，我们将在之后的`in depth`关于ES6 modules的文章探讨。
