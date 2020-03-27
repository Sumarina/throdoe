---
title: nodejs从入门到放弃（三）
toc: true
comments: true
copyright: true
date: 2018-12-20 14:25:19
tags: ["nodejs"]
categories: "程序员"
---
如果读过了我的从入门到放弃的第二篇，就会对Node的异步I/O有所了解，接下来我要说的是Node中还存在一些与异步I/O无关的异步API。
******
## 定时器
`setInterval()`和`setTimeout()`与浏览器中的API是一致的，分别用于单次和多次定时执行任务。他们实现的原理与异步I/O毕竟类似，只是没有线程池的参与。
调用`setInterval()`huo3`setTimeout()`创建的定时器会被插入到定时器观察者内部的红黑树中，每次Tick执行，就会从红黑树中取出定时器对象，检查是否超过定时时间，如果超过，则形成一个事件，回调函数立即执行。看图：
![定时器](./定时器.png)
## process.nextTick()
由于事件循环的特点，定时器的精确度是不够的，比如某个任务耗时许久，下个任务原本已经到了执行事件却因为这个耗时任务而不能立即执行。再加上定时器是动用红黑树，创建定时器对象和迭代等操作，毕竟浪费性能。相对而言，`process.nextTick()`的操作相对较为轻量，每次调用`process.nextTick()`都是将回调函数放入队列，下一轮tick取出执行。
## setImmediate()
`setImmediate()`也是将回调函数延迟执行，在Nodev0.9.1之前，`setImmediate()`还没实现呢，是通过`process.nextTick()`来完成。
```
process.nextTick(function(){
    console.log("延迟执行");
});
console.log("正常输出");
//result
正常输出
延迟执行
```
```
setImmediate(function(){
    console.log("延迟执行");
});
console.log("正常输出");
//result
正常输出
延迟执行
```
从以上两段代码看，结果一样，但实际它们还是有区别的。
```
setImmediate(function(){
    console.log("setImmediate延迟执行");
});
process.nextTick(function(){
    console.log("nextTick延迟执行");
});
console.log("正常输出");
//result
正常输出
nextTick延迟执行
setImmediate延迟执行
```
从结果可以看出`process.nextTick()`的优先级高于`setImmediate()`，是因为*事件循环对观察者的检查是有先后顺序的*（ps:继续往下看会详细写这块内容。）
`process.nextTick()`的回调函数是存在一个数组，`setImmediate()`的结果是保存在链表中。
在行为上，`process.nextTick()`在每轮循环会将数组中的回调函数全部执行完，`setImmediate()`在每轮循环中执行链表中的一个回调函数。用一段代码来证明：
```
process.nextTick(function(){
    console.log("next tick....1");
});

process.nextTick(function(){
    console.log("next tick....2");
});

setImmediate(function(){
    console.log("immediate.....1");
    process.nextTick(function(){
        console.log("next tick....3");
    });
})

setImmediate(function(){
    console.log("immediate.....2");
})
//result
next tick....1
next tick....2
immediate.....1
next tick....3
immediate.....2
```
从结果可以看出第一个`setImmediate()`的回调函数执行后，并没有立即执行第二个，而是进入下一轮循环，再次优先调用`process.nextTick()`。
以上便是在Node中的几种不涉及异步I/O的异步API。但这几种API具体在事件循环中又是一个什么执行顺序？继续往下看，please。
先来一段分割线。。。
******
## Event Loop
当Node启动时，便会初始化event loop，每一个event loop都包含六个循环阶段。看下图：
![event loop循环阶段](./event loop循环阶段.png)
* *timers阶段*：在这个阶段执行定时器预定的callback，比如`setTimeout(callback)`和`setInterval(callback)`。
* *I/O callbacks阶段*：除其他阶段以外的callback。
* *idle，prepare阶段*：仅node内部使用。
* *poll阶段*：获取新的I/O事件，适当条件下阻塞。
* *check阶段*：执行`setImmediate()`设定的callback。
* *close callback阶段*：close的callback。
每一个阶段都有各自相对应的队列，当event loop运行到指定阶段，node将从对应的队列取出callback执行，当队列中的callback执行完或者超过该阶段的上限，event loop会转入下一个阶段。
***注意:`process.nextTick()`不在上面任何一个阶段。***
### setTimeout and setImmediate
`setTimeout`在poll阶段空闲，且设定事件到达后执行，在timer阶段执行。
`setImmediate`在poll阶段完成后进入check执行。
二者调用顺序取决event loop上下文，在异步I/O callback之外调用，执行顺序不确定。
```
setTimeout(() => {
    console.log("setTimeout finished...");
});
setImmediate(function(){
    console.log("setImmediate finished...");
})
//result
setTimeout finished...
setImmediate finished...
也可能出现
setImmediate finished...
setTimeout finished...
```
在异步I/O callback中的调用顺序一定是先执行`setImmediate`后执行`setTimeout`：
```
fs.readFile("./main/read.txt", () => {
    //约5ms读取完毕
    console.log("read file finished...");
    setTimeout(() => {
        console.log("setTimeout finished...");
    });
    setImmediate(function () {
        console.log("setImmediate finished...");
    })
})
//result
read file finished...
setImmediate finished...
setTimeout finished...
```
是因为异步I/O callback是在poll阶段执行，执行完毕后进入到check阶段执行`setImmediate`，后进入到timer阶段执行`setTimeout`。
### Poll阶段
poll阶段是整个event loop比较重要的阶段。
在node.js中，任何异步方法（除timer，closer，setImmediate）完成时，都会将其callback加到poll queue，并立即执行。
当event loop进入poll阶段，分两种情况：
1. *有timer*：在有timer的情况下，如果poll的队列中没有任何待处理的事件，就会检查timer队列中有没有已经到了时间需要执行的事件，如果至少有一个，那么event loop将按照循环顺序进入timer阶段执行timer队列。
2. *没有timer*：在没有timer的情况下，如果poll队列中有待处理的事件，则依次执行，直到队列为空，或者执行的callback到达系统上限。当poll队列为空，则会判断是否有`setImmediate()`设置的callback，如果存在则结束poll阶段，马上进入check阶段执行check队列中的事件；如果没有`setImmediate()`设置的callback，则event loop会阻塞在poll阶段，死等，直到有callback加入队列。
看下面两段代码：
```
fs.readFile("./main/read.txt",()=>{
    //约5ms读取完毕
    console.log("read file finished...");
})
setTimeout(() => {
    console.log("setTimeout finished...");
});
setImmediate(function(){
    console.log("setImmediate finished...");
})
//result
setTimeout finished...
setImmediate finished...
read file finished...
```
在执行到poll阶段的时候，已经有设置过的timer，而且此时poll阶段的队列为空，而timer的队列中已经有待执行的callback，则马上结束poll阶段按顺序执行进入到timer阶段，这里有`setTimeout(callback)`和`setImmediate(callback)`的回调顺序取决event loop上下文确定，也就是说它俩的执行先后顺序不确定。多执行几次说不定会出现先执行`setImmediate(callback)`的回调事件。
```
fs.readFile("./main/read.txt",()=>{
    //约5ms读取完毕
    console.log("read file finished...");
})
setTimeout(() => {
    console.log("setTimeout finished...");
},6);
setImmediate(function(){
    console.log("setImmediate finished...");
})
//result
setImmediate finished...
read file finished...
setTimeout finished...
```
在执行到poll阶段的时候，此时文件还未读取完毕，所以poll阶段的队列为空，虽然已经有设置过的timer，时间未到，timer队列中没有待处理的事件，此时有`setImmediate(callback)`设置的回调，则马上结束poll阶段进入check阶段执行callback，该轮event loop结束后进入到下一次tick，timer队列依旧为空，进入到poll阶段，此时文件读取完毕，回调函数被当作事件进入到poll队列，执行该事件后，timer中的callback已经到时间执行，则马上按顺序进入到timer阶段执行队列中的事件。
### process.nextTick()
之前提到`process.nextTick()`不属于任何一个阶段，那它什么时候执行？
`process.nextTick()`是在各个阶段切换的中间执行，也就是说是从一个阶段切换到下一个阶段这个间隙执行。
```
fs.readFile("./main/read.txt",()=>{
    //约5ms读取完毕
    console.log("read file finished...");
})
setTimeout(() => {
    console.log("setTimeout finished...");
});
setImmediate(function(){
    console.log("setImmediate finished...");
})

process.nextTick(function(){
    console.log("next tick....1");
});

process.nextTick(function(){
    console.log("next tick....2");
});
//result
next tick....1
next tick....2
setTimeout finished...
setImmediate finished...
read file finished...
```
上面代码中从timer阶段进入下个阶段间隙执行`process.nextTick()`，上面提到过`process.nextTick()`的回调函数是保存在数组中，一次性取出全部执行。
`process.nextTick()`是早起Node版本无`setImmediate()`时的产物，node作者推荐尽量使用`setImmediate()`。
这是为啥子呢？
试想，如果我们在一个递归中无限循环调用`process.nextTick()`，那是不是其他阶段的callback则没有机会执行了呢？
而`setImmediate()`就不一样，只在check阶段，而且每次从队列中只取出一个事件执行，那这样大家都有机会执行了，能分到一杯羹。。ps:啊哈哈哈,不知道为啥此刻脑子想到某个港剧中几个大佬抢地盘的场景。
以上所有便是Node中事件循环的整个过程。
******
ps:参考朴灵的《深入浅出》和 https://cnodejs.org/topic/57d68794cb6f605d360105bf












