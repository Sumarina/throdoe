---
title: 关于JavaScript类型转化
toc: true
comments: true
copyright: ture
date: 2018-11-13 10:47:44
tags: ['javascript']
categories: "程序员"
---
JavaScript类型转化分隐式转化和显式转化。   
在JavaScript中类型转化只能转化为三种基本类型：   
1. to string
2. to number
3. to boolean

******
##  to string
所有调用String显式转化为string类型的都可以得到你想要的结果。
当遇到操作符+号，其中一个操作数为string类型，另一个操作数则会隐式转化为string类型。
```
String(1) //显式转换 '1'
String(1.2) //显式转换 '1.2'
String(null) //显式转换 'null'   
String(undefined) //显式转换 'undefined'  
String(true) //显式转换 'true'
String(false) //显式转换 'false'
1+'' //隐式转换  操作符+号 且其中一个操作数为string类型。结果'1'
```
## to number
调用Number显式转化为number类型。
隐式转化的情况有以下几种：
- 比较操作符（<,>,<=,>=）
- 按位操作符（| & ^ ~）
- 四则运算 （- + * / %） 其中操作符为+号，一方操作数为string类型，则不会隐式转化为number类型。
- 一元操作符 （+） 
- loose equality operator == ，也包括!= （不知道loose equality应该翻译成啥样的词。大意就是判断是否相等，只比较值，不考虑type是否相等。）
```
Number('') // 0
Number(null) //  0
Number(undefined) // NaN   
Number('12') // 12   
Number('12s') // NaN  
Number(true) // 1
Number(false) // 0   
```
**注意：虽然`Number(null)`为0，但`null==0`为false，`null==undefined`为true。NaN不等于任何值，包括它自己**

## to boolean
调用Boolean显示转化为boolean，或者遇到逻辑操作符(|| && !) 则会隐式转化为boolean类型。
除了以下几种情况转为false，其他则为true。
```
Boolean('')  // false 
Boolean(0)  // false 
Boolean(-0)  // false 
Boolean(null) // false
Boolean(undefined) // false
Boolean(NaN) // false    
Boolean(false) // false 
```
即使是[]或者{}对象，也会转化为true
```
Boolean({})             // true
Boolean([])             // true
Boolean(function() {})  // true
```
以上为基本类型显式或隐式转化。

******
## Object convert to Number or String 
Object 依然只能转化为number、string和boolean 三种基本类型。
- Object转化为boolean最简单，永远为true.即使为空对象或者空数组等。
- Object转化为stirng或者number，通过调用一个内置方法`[[ToPrimitive]]`实现转化。代码如下：
```
function ToPrimitive(input, preferredType){
  
  switch (preferredType){
    case Number:
      return toNumber(input);
      break;
    case String:
      return toString(input);
      break
    default:
      return toNumber(input);  
  }
  
  function isPrimitive(value){
    return value !== Object(value);
  }

  function toString(){
    if (isPrimitive(input.toString())) return input.toString();
    if (isPrimitive(input.valueOf())) return input.valueOf();
    throw new TypeError();
  }

  function toNumber(){
    if (isPrimitive(input.valueOf())) return input.valueOf();
    if (isPrimitive(input.toString())) return input.toString();
    throw new TypeError();
  }
}
```
一般来说，转化的过程如下：
1. input为基本类型，啥也不做直接return。
2. 调用input.toString(),结果为基本类型，return。
3. 调用input.valueOf()，结果为基本类型，return。
4. 不满足以上三点则抛出TypeError.
转化为number，则先调用valueOf(),再调用toString();转化为string，则正好相反，先toString(),再valueOf()。
不同操作符会触发不同转化规则，根据参数preferredType要么转化为number要么是string。
**注意：遇到==和+操作符，preferredType未赋值或者等于默认值时，则默认转为number，除了Date。**   

```
var obj = {
  prop: 101,
  toString(){
    return 'Prop: ' + this.prop;
  },
  valueOf() {
    return this.prop;
  }
};

console.log(String(obj));  // 'Prop: 101'
console.log(obj + '')      // '101'
console.log(+obj);         //  101
console.log(obj > 100);    //  true

/** Date测试 */
let d=new Date();
let str=d.toString();
let num=d.valueOf();
console.log(d==str) //true
console.log(d==num) //false
```  
******
参考链接：https://medium.freecodecamp.org/js-type-coercion-explained-27ba3d9a2839





