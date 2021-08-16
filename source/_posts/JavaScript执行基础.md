---
title: JavaScript基础
date: 2021-08-16 15:39:43
subtitle:
categories: 基础
tags: JavaScript基础
cover: /images/5af17f7f881b11ebb6edd017c2d2eca2.jpg
---


## 执行上下文
JS是单线程的，执行顺序从上到下

### 执行机制
- 先编译，再执行
- 编译阶段，变量和函数会被存放到变量环境中，变量的默认值会被设置为 undefined
- 代码执行阶段，JavaScript 引擎会从变量环境中去查找自定义的变量和函数

### 变量提升
指在 JavaScript 代码执行过程中，JavaScript 引擎把变量的声明部分和函数的声明部分提升到代码开头的“行为”。变量被提升后，会给变量设置默认值，这个默认值就是我们熟悉的 undefined。

```javascript
var myname = '极客时间'

// 等同于 ==>
var myname    //声明部分
myname = '极客时间'  //赋值部分
```
![20210816161935.png](https://i.loli.net/2021/08/16/zxCZsK9i1GEyHDR.png)

关于`同名变量和函数`的两点处理原则：
1. 如果是同名的函数，JavaScript编译阶段会选择最后声明的那个。
2. 如果变量和函数同名，那么在编译阶段，变量的声明会被忽略。 

```javascript
// 变量和函数同名，变量的声明被忽略。
showName() // 1
var showName = function() {
  console.log(2)
}
function showName() {
    console.log(1)
}
```

注意：`在块级作用域中定义的函数，在全局上下文中只是进行函数名变量进行提升，但是不执行赋值操作，所以为 undefined`

```javascript
function foo(){
  console.log(g) // undefined 
  if(true){
      console.log('hello world');
      function g(){ return true; }
  }
}
// ===> 等价于
function foo(){
  if(true){
      console.log("hello world");
      var g = function(){return true;}
  }
}
```


## 调用栈（call stack）
JS 引擎通常把用来管理执行上下文的栈称为执行上下文栈，又称`调用栈`。

调用栈是 JavaScript 引擎追踪函数执行的一个机制，当一次有多个函数被调用时，通过调用栈就能够`追踪到哪个函数正在被执行`以及各函数之间的`调用关系`。

### 利用浏览器查看调用栈的信息
```javascript
var a = 2
function add(b,c){
  return b+c
}
function addAll(b,c){
var d = 10
result = add(b,c)
return  a+result+d
}
addAll(3,6)
```
![c0d303a289a535b87a6c445ba7f34fa2.png](https://i.loli.net/2021/08/16/oz7ntjXHSqVKmb6.png)

可以使用`console.trace()`可以看到当前的函数调用关系

### 栈溢出
`调用栈是有大小的`。栈的深度根据个人的机器环境不同
```javascript
// 栈溢出 
function runStack (n) {
  if (n === 0) return 100;
  return runStack( n- 2);
}
runStack(100000) // Uncaught RangeError: Maximum call 
```
![20210816171641.png](https://i.loli.net/2021/08/16/qESsvKWRNJ3fbcz.png)

## 作用域（scope）

- `全局作用域`中的对象在代码中的任何地方都能访问，其生命周期伴随着页面的生命周期
- `函数作用域`就是在函数内部定义的变量或者函数，并且定义的变量或者函数只能在函数内部被访问。函数执行结束之后，函数内部定义的变量会被销毁

![5af17f7f881b11ebb6edd017c2d2eca2.jpg](https://i.loli.net/2021/08/16/6keCLBX3oNs9Vif.jpg)
