---
title: JAVA 对象得生命周期
date: 2013/8/15 20:47:30
categories:
- Shit Done
tags:
- java 
- 对象的生命周期
---

### **一.垃回收器判断java对象死活的算法**

堆中几乎存放着java 世界的所有对象，垃圾回收器在对堆回收前，第一件事情就是判断对象那些是“活”的，哪些是“死”的。那判断的算法是什么呢？
#### **1. 引用计数算法**

引用计数算法是对每个对象配置一个引用计数器，当一个地方引用它时，它的引用计数器就加1，当引用失效后，计数器减1.当计数器的值为0时，就表示这个对象不可用了，也即是“死”了。

引用计数算法的实现很简单，判定效率很高，很多时候都是一个不错的算法。

<!--more-->
#### **2. 根搜索算法**

根搜索算法基本思路就是通过一系列称为 “GC Roots” 的对象作为起始点，从这些节点向下搜索，搜索走过的路径称为“引用链（Refenrence Chain）”，当一个对象到所有的GC Roots都没有引用链的时候，则证明这个对象是不可用的。则将会被判断为可回收的对象。如下图中的对象5， 6， 7，虽然它门之间有关联，但是它门与GC Roots之间没有引用链，所以在垃圾回收的时候是会被认为是可回收的。

{% img ../images/GC_Roots.png %}

那java中到底使用了是哪种算法呢，我们通过一段代码来验证：

``` java

    public class ReferenceCountingGC {
        public Object instance = null;

        private static final int _1MB = 1024 * 1024;

        //这个成员设计目的是为了占内存，以便能在GC日志中看到清楚是否被回收过
        private byte[] bigSize = new byte[2 * _1MB];

        public static void testGC() {
            ReferenceCountingGC objA = new ReferenceCountingGC();
            ReferenceCountingGC objB = new ReferenceCountingGC();

            //让它门相互引用,满足了算法一
            objA.instance = objB;
            objB.instance = objA;

            //置空它们，满足了算法二
            objA = null;
            objB = null;

            //开启回收
            System.gc();
        }

        public static void main(String[] args) {
            ReferenceCountingGC.testGC();
        }
    }
```

看GC日志（eclipse中gc日志输出设置，参见：http://www.myexception.cn/eclipse/1268020.html）

{% img ../images/gclog.png %}

可以看到两个对象4M（`4612K->375K`）左右被回收了，也就是数说从侧面证明了java不是使用引用计数算法，而是根搜索算法。这是因为引用计数算法无法解决对象之间相互循环的引用的问题。顺便提一句python是使用了计数引用算法。
### **二. 对象的真正审判**
#### **1. java中的引用**

无论是以上两种的哪种算法，都是依靠引用来判断的。在JDK1.2之后，java对引用的概念进行了扩充，将引用分为：
- 强引用：指代码中最常见的Object obj = new Object()这种引用，只要强引用存在这个对象就永远不会被回收掉。
-  软引用：用来描述一些还有用,但是并非必须的对象。对于软引用关联着对象，在系统将要发生内存溢出之前，将这些对象列入回收的范围，进行第二次回收，但是回收后还不够内存的话，才会抛出溢出异常。JDK1.2之后，提供了SoftReference类来实现软引用。
-  弱引用：用来描述一些非必须的对象，相对软引用的强度较弱。被弱引用关联的对像只能活到下次垃圾收集发生之前，无论当前内存是否足够。
-  虚引用： 也称为幽灵引用，虚引用关联的对象与对象的生命周期无关，也不能通过虚引用获取对象，只是能在回收的时候接受到一个系统通知。
#### **2. 存活还是死亡**

在经过了根搜索算法无法到达后，基本这个对象被定为了死刑。但是也不是绝对的。
垃圾收集器对所有无法达到的对象进行一次标记和筛选，筛选出需要执行`finalize（）`的对象。

判断一个对象`无需`执行`finallize（）`方法的标准是：这个对象没有重写fianlize（）方法或者finalize（）方法被虚拟机调用过了。这些无需执行`finalize（）`方法的对象将会被直接回；而需要执行`finalize（）`的对象将在执行完后再判断是否要回收，也就是`fianlize（）`是对象的最后一棵救命的稻草。可以从下面的代码验证：

``` java

    public class FinalizeEscapeGC {
    public static FinalizeEscapeGC SAVE_HOOK = null;

    public void isAlive(){
        System.out.println("yes, I am still alive :)");
    }


    @Override
    protected void finalize() throws Throwable {
        super.finalize();
        System.out.println("finalize method executed...");
        FinalizeEscapeGC.SAVE_HOOK = this;   //再finalize（）方法中拯救自己
    }

    public static void  main(String[] args) throws InterruptedException{
        SAVE_HOOK = new FinalizeEscapeGC();

        //置空SAVE_HOOK,然后调用回收机制，第一次调用finalize（），发现没有被回收。
        SAVE_HOOK = null;
        System.gc();

        Thread.sleep(500);
        if(SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        } else{
            System.out.println("no, I am dead :( ");
        }

        //第二次执行一样的代码，但是却被回收了。
        SAVE_HOOK = null;
        System.gc();

        Thread.sleep(500);
        if(SAVE_HOOK != null){
            SAVE_HOOK.isAlive();
        } else{
            System.out.print("no, I am dead :( ");
        }

    }
}

```

代码的输出为：

``` java
        finalize method executed...
        yes, I am still alive :)
        no, I am dead :( 
```

可以看到第一次垃圾回收的时候执行了`finalize()`方法，而方法体中对对象进行了拯救，所以对象没有被回收，但是第二次垃圾回收到来的时候，由于`fianlize（）`已经被执行过一次了，所以不会被执行，这就是对象被回收的原因。
