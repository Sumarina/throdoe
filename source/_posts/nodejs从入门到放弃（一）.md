---
title: nodejs从入门到放弃（一）
toc: true
comments: true
copyright: true
date: 2018-11-15 11:30:45
tags: ["nodejs"]
categories: "程序员"
---
年初就买了一本《深入浅出node.js》，一直拖到现在才看，为自己的拖延症汗颜。
不为自己懒惰找借口。
*****
## CommonJs的规范
1. 通过require引用模块
2. 模块定义：一个文件就是一个模块
3. 模块标识：传递给require方法的参数名必须是小驼峰命名的字符串或相对路径、绝对路径。
*****
## NodeJs如何实现CommonJs规范?
先看下面这段代码，有没有考虑过为什么直接可以调用require方法？为什么只有通过exports属性导出方法？module是什么？module和exports是什么关系？
```
var fs=require('fs');
// console.log(module.paths);
//模块定义
function read(){
    fs.readFile('readme.md',function(err,file){
        console.log('read finished...');
    })
    console.log('read start...');
}
function write(){
    console.log("write");
}

module.exports.readFiles=read;
module.exports.writeFiles=write;
```
实际上，在编译过程中，Node对获取的js文件内容会有一个封装，怎么封装？就是在内容头部加`(function(require,exports,module,__dirname,__filename){/n`,尾部会加上`})/n`。也就是说我们能用的require,exports,module,__dirname,__filename都是通过参数传递进来的。封装后，每个模块都有了隔离的作用域。
包装之后的代码通过原生模块vm的runInThisContext()方法得到一个具体的function对象，把当前文件的exports,module,require，路径以及完整路径传递给这个function，返回exports以及挂在这个属性的方法和属性。所以我们只能访问到exports上的属性和方法，其余未挂在这个对象上的则不能访问。
看下面这段代码：
```
function Module(id, parent) {
    this.id = id;
    this.exports = {};
    this.parent = parent;
    if (parent && parent.children) {
        parent.children.push(this)
    }
    this.filename = null;
    this.loaded = false;
    this.children = [];
}
```
在Node中每个文件都是一个对象。Module就是模块本身。从上面代码不难看出exports是module的属性，这意味着我们可以用`module.exports.readFiles=read;`或者`exports.readFiles=read;`的方式导出方法或属性。
可能又会有好奇宝宝问为什么用`exports.readFiles=read`的方式，而不是`exports=read`的方式？因为exports是作为形参传入，直接赋值形参是不会改变方法以外的exports的内容。来，看看下面这段代码有木有些许明白了呢？
```
var a=10;
function change(a){
    a=20;
}
change(a);
console.log(a) //10
```
以上奏是NodeJs对CommonJs规范的实现。
*****
ps:参考朴灵的《深入浅出》
