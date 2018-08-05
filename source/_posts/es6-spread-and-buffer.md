---
title: (译)ES6 Spread and Butter in Depth
date: 2015/10/30 12:01:25
categories:
- Shit Done
tags:
- ES6
- Spread
- Buffer
---
> [原文地址](https://ponyfoo.com/articles/es6-spread-and-butter-in-depth#rest-parameters)

我们在讨论[解构]()的时候已经涉及到了一部分今天的内容。这篇将比其他的内容短一些，因为简单的特性没有太多可以说的。尽管如此，就像我在[ES6 in Depth](https://ponyfoo.com/articles/tagged/es6-in-depth)系统中说的，最简单的特性往往也[最有用](https://github.com/rockcoder23/blog.fexnotes.com/issues/24)，让我们来看看把。
### Rest 参数

你知道有时候函数有一大堆的参数，你最终不得不用`arguments` 魔法变量来处理他们。考虑下面这样把任意传入的参数的链接为字符串的方法。

``` js
    function concat() {
        return Array.prototype.slice.call(arguments).join(' ')
    }
    var result = concat('this', 'was', 'no', 'fun')
    console.log(result)  //<- 'this was no fun'
```

rest参数语法通过添加`...`在参数名的前面能够让你生成一个包含`function`参数的真实的`Array` 。语法十分的简单，事实上一个真实的数组使用起来也十分的方便。对于我来说我很高兴再也不需要借助于`arguments`了。

``` js
    function concat (...word) {
     return word.join(' ')
    }
    var result = concat('this', 'is', 'okay')
    console.log(result); //<- 'this is okay'
```

当你的`function`有更多的参数的时候略微有一些不同。不管什么时候我声明一个带有rest参数的方法的时候，我会考虑下面几点。
- Rest 参数将包含所有调用函数的`arguments`。
- 每次在函数的左边添加一个参数，对于Rest参数形成的数组来说就像调用了rest.shift()。
- 注意你不允许把参数放到Rest参数的右边，Rest参数只能是最后一个参数。

要形象化的描述这个最好是通过实践，而不是一句话的总结。下面这个方法将计算除了第一个参数(用来乘以后面的参数和)，剩余参数的和。为了不重复的调用函数，`.shift()`移除并且返回数组中的第一个参数，把函数变得类似一个有记忆的设备一样。

``` js
    function sum(){
        var number = Array.prototype.slice.call(arguments) //numbers gets all arguments
        var multiplier = numbers.shift()
        var base = numbers.shift()
        var sum = numbers.reduce((accumulator, num) => accumulator + num, base) 
        return multiplier * sum
    }
    var total = sum(2,  6,  10,  8,  9)
    console.log(total)  //<- 66
```

下面是我们使用rest 参数改造之后的函数的样子，注意我们没有使用`arguments` 也没有再使用shifting。 这将相当赞因为这极大的减小我们函数的复杂度，这样我们将能更专注于它本身的功能而不是更多在处理`arguments`。

``` js
    function sum(multiplier, base, ...numbers) {
        var sum = numbers.reduce((accumulator, num) => accumulator + num, base)
        return multiplier * sum
    }
    var total = sum(2, 6, 10, 8, 9)
    console.log(total) //<- 66
```
### 扩展运算符

通常你通过传递参数来调用一个方法。

``` js
    console.log(1, 2, 3) //<- '1 2 3'
```

有时你有一些参数是放在一个列表中但是你又不想为了一次函数调用而使用索引来获取，或者这个列表是动态的生产的你根本不能这样获取。所以你一般会使用`.apply()`。这让你觉得不是很方便，因为`.apply`需要指定一个上下文`this`,而你得传入一个或许就不相关的对象(类似 `null`)。

``` js
    console.log(console, [1, 2, 3])  //<- '1, 2, 3'
```

扩展运算符可以被当做`.apply`的替代品(原文：butter knife, 黄油刀 Orz..) 。也不需要指定上下文，你只需要在数组前面使用`...` ，就像rest参数一样。

``` js
    console.log(...[1, 2, 3]) //<- '1, 2, 3'
```

其实扩展运算符可以用在任何事先iterable接口的对象上，这在我们下周一将要讨论的`iterators`特性中会有更好的补充，甚至能作用在`document.querySelectorAll('div')`的结果。

``` js
    [...document.querySelectorAll('div')] //<-[<div>, <div>, <div>]
```

另一个更好的方面是你可以混合搭配正常的参数，并且他们可以按你预期的那样执行。这在你处理有很多的参数的es5代码时非常有用。

``` js
        console.log(1, ...[2, 3, 4], 5) //becomes `console。log(1，2，3，4，5) //<- '1 2 3 4 5'
```

是时候讨论个真实的例子了.我有时在Express中使用下面的方法让[`morgan`](https://github.com/expressjs/morgan)(记录Express中的请求日志)通过[`winston`](https://github.com/winstonjs/winston)列化它的数据，为了实现多点传输日志服务。我把换行符从字节流中移除并且赋给message，因为winston已经处理过了，而且我还把一些元数据包括进程id和ip放在了`arguments`中，最后我使用`.apply`来调用winston的处理方法。 如果你仔细查看下面的方法，你会发现真正起作用的是我黄色高亮的那句，而且其他的都是在处理`arguments`.

``` js
    function createWriteStream (level) {
        return {
            write: function(){
                var bits = Array.prototype.slice.call(arguments)
                var message = bits.shift().replace(/\n+$/， '')
                bits.unshift(message)
                bits.push({hostname: os.hostname(), pid: process.pid})
                winston[level].apply(winston, bits)   //上文提到的黄色高亮语句指得是这句
            }
        }
    }
    app.use(morgan(':status :method :url', {
        stream.createWriteStream('debug')
    }))
```

我们可以使用ES6来彻底简化这个代码，首先，我们可以使用rest 参数来代替`arguments`。rest参数已经给我们一个现成的数组，所以将没有变相的调用。我们可以提取message作为第一个参数，而且我们可以直接调用`winston[level]`通过结合正常参数和`...bits`。这时代码就变得有模有样了，而且每一部分都是跟你想要实现的功能相关，比如改了调用`winston[level]`调用的参数。这代码相比于之前手动维护arguments来说，实现了自我的救赎[原文：a battle of wits against JavaScript itself]。

``` js

    function createWriteStream(level) {
        return {
            write: function (message, ..bits) {
                winston[level](message.replace(/\n+$/, ''), ...bits, {
                    hostname: os.hostname(), pid: process.pid
                })
            }
        }
    }
```

另一个方法再次精简我们的代码使用箭头函数. 但是在这个场景下，将会是代码理解起来更复杂。你可以使用`msg` 代替`message` 来实现让整个代码在一行，然后使用rest参数和扩展运算符来调用`winston[level]`，让它不论是你还是你的队友在一周之后再看这个代码不思考15分钟都难以理解这个方法的作用。

``` js
    var proc = {hostname: os.hostname(), pid: process.pid}
    function createWriteStream (level) {
        return {
            write(msg, ...bits) => winston[level](msg,replace(/\n+$/, ''), ...bits, proc)
        }
    }
```

保持上一版本的写法还是比较明智的选择，然而这很好的证明了在这个情形下使用箭头函数只会增加复杂性。在其他场景下可能不会。这完全取决于你，所以你得理智的使用ES6的特性，到底是为了提高你的代码的可维护性还是只是为了使用ES6而牺牲了代码的可维护性。

下面列举了一些有用的用法，显然你可以使用扩展运算符来创建一个数组，但是你也使用解构，这有点类似...rest原理。有一个不是很常见但是很值得一提的场景是你可以使用扩展运算符来代替`apply`即使是使用`new`运算符的时候。

| 场景 | ES5 | ES6 |
| --- | --- | --- |
| Concatenation | [1, 2].concat(more) | [1, 2, ...more] |
| Push onto list | list.push.apply(list, [3, 4]) | list.push(...[3, 4]) |
| Destructuring | a = list[0], rest = list.slice(1) | [a, ..rest] = list |
| new + apply | new (Date.bind.apply(Date, [null, 2015, 31, 8])) | new Date(... [2015, 31, 8]) |
### 默认赋值操作

默认赋值操作我们在解构那边文章已经涉及到了，但是只是简单的介绍了下。就像你可以在解构中使用默认值一样，你可以为函数的任何参数指定一个默认值，就像下面展示的那样。

``` js
    function sum (left = 1, right = 2) {
        return left + right 
    }
    console.log(sum()) //<- 3
    console.log(sum(2)) //<- 4
    console.log(sum(1, 0)) //<- 1
```

考虑[`dragula`](https://github.com/bevacqua/dragula/blob/f5f4c569780b0db160269e978eaf69dc36e421bb/dragula.js#L27-L37)里面的初始化options的代码。

``` js
    function dragula (options) {
        var o = options || {};
        if (o.moves === void 0) { o.moves = always;}
        if (o.accepts === void 0) { o.accepts = always; }
        if (o.invalid === void 0) { o.invalid = invalidTarget; }
        if (o.containers === void 0) { o.containers = initialContainers || []; }
        if (o.isContainer === void 0) { o.isContainer = never; }
        if (o.copy === void 0) { o.copy = false; }
        if (o.revertOnSpill === void 0) { o.revertOnSpill = false; }
        if (o.removeOnSpill === void 0) { o.removeOnSpill = false; }
        if (o.direction === void 0) { o.direction = 'vertical'; }
        if (o.mirrorContainer === void 0) { o.mirrorContainer = body; }
    }
```

你认为是否值得使用ES6的默认赋值操作语法来重构这部分代码？你会怎么重构？
