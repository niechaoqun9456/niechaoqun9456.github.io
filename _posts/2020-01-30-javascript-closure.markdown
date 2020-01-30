---
layout: post
title:  "JavaScript Closure"
date:   2020-01-30 18:02:10 +0800
categories: jekyll update
---

### What Is Closure
> A closure is the combination of a function bundled together (enclosed) with references to its surrounding state (the lexical environment). In other words, a closure gives you access to an outer function’s scope from an inner function. In JavaScript, closures are created every time a function is created, at function creation time.

闭包是函数和函数创建时的词法环境的组合。换句话说，闭包使内层函数可以访问内层函数的作用域。每个函数的创建都形成一个闭包，在函数创建时形成，而非函数调用时。

### Lexical Scoping词法作用域
```javascript
function init() {
  var name = 'Mozilla'; // name is a local variable created by init
  function displayName() { // displayName() is the inner function, a closure
    alert(name); // use variable declared in the parent function
  }
  displayName();
}
init();
// alert 'Mozilla'
```
这是一个词法作用域的例子，展示了词法解析器在函数嵌套时如何解析变量：内层函数可以访问声明在外层函数中的变量，变量声明的地点决定了它的作用域（在哪里可以访问）。

这里displayName函数创建时形成的闭包就是displayName函数本身以及displayName创建时的词法环境：该词法环境中包含了name = 'Mozilla'这个变量。

```javascript
function makeFunc() {
  var name = 'Mozilla';
  function displayName() {
    alert(name);
  }
  return displayName;
}
var myFunc = makeFunc();
myFunc();
```

调用makeFunc时创建了displayName函数，displayName函数维护了一个指向其词法环境的引用，该词法环境中包含了name变量。所以当myFunc调用时，name因为在displayName的词法环境中所以是可访问的。

```javascript
function makeAdder(x) {
  return function(y) {
    return x + y;
  };
}
var add5 = makeAdder(5);
var add10 = makeAdder(10);
console.log(add5(2));  // 7
console.log(add10(2)); // 12
```
add5和add10都是闭包，他们的词法环境中都包含了x变量，add5的词法环境中x=5,add10的词法环境中x=10

### 用闭包模仿私有方法
```javascript
var counter = (function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  };
})();
console.log(counter.value()); // logs 0
counter.increment();
counter.increment();
console.log(counter.value()); // logs 2
counter.decrement();
console.log(counter.value()); // logs 1
```
privateCounter变量和changeBy函数在匿名函数外都无法访问，但是可以通过increment、decrement、value形成的闭包可以访问到。increment、decrement、value三个闭包共享同一个词法环境（包含privateCounter变量和changeBy函数）。

```javascript
var makeCounter = function() {
  var privateCounter = 0;
  function changeBy(val) {
    privateCounter += val;
  }
  return {
    increment: function() {
      changeBy(1);
    },
    decrement: function() {
      changeBy(-1);
    },
    value: function() {
      return privateCounter;
    }
  }
};
var counter1 = makeCounter();
var counter2 = makeCounter();
alert(counter1.value()); /* Alerts 0 */
counter1.increment();
counter1.increment();
alert(counter1.value()); /* Alerts 2 */
counter1.decrement();
alert(counter1.value()); /* Alerts 1 */
alert(counter2.value()); /* Alerts 0 */
```
对比上面那个例子，couter1和conter2形成的闭包，互不干扰，各自词法环境内的变量的修改互不影响。

### Closure Scope Chain闭包作用域链
每个闭包都有三个作用域
- Local Scope (Own scope) 本地作用域
- Outer Functions Scope 外层函数作用域
- Global Scope 全局作用域
```javascript
// global scope
var e = 10;
function sum(a){
  return function(b){
    return function(c){
      // outer functions scope
      return function(d){
        // local scope
        return a + b + c + d + e;
      }
    }
  }
}
console.log(sum(1)(2)(3)(4)); // log 20
// We can also write without anonymous functions:
// global scope
var e = 10;
function sum(a){
  return function sum2(b){
    return function sum3(c){
      // outer functions scope
      return function sum4(d){
        // local scope
        return a + b + c + d + e;
      }
    }
  }
}
var s = sum(1);
var s1 = s(2);
var s2 = s1(3);
var s3 = s2(4);
console.log(s3) //log 20
```

### 在循环中创建闭包：一个常见错误
```html
<p id="help">Helpful notes will appear here</p>
<p>E-mail: <input type="text" id="email" name="email"></p>
<p>Name: <input type="text" id="name" name="name"></p>
<p>Age: <input type="text" id="age" name="age"></p>
```
```javascript
function showHelp(help) {
  document.getElementById('help').innerHTML = help;
}
function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    var item = helpText[i];
    document.getElementById(item.id).onfocus = function() {
      showHelp(item.help);
    }
  }
}
setupHelp();
```
三个输入框的提示都是Your age (you must be over 16)

因为这里item的声明用了var(var声明的变量的作用域是整个封闭函数)，赋值给onFocus的function形成的3个闭包的词法环境中的item最后都被第3次赋值覆盖，定格在'Your age (you must be over 16)'。

最快的改法是用let（块级作用域，每个循环都有一个）替换var，另外也可以用闭包解决这个问题：
```javascript
function setupHelp() {
  var helpText = [
      {'id': 'email', 'help': 'Your e-mail address'},
      {'id': 'name', 'help': 'Your full name'},
      {'id': 'age', 'help': 'Your age (you must be over 16)'}
    ];

  for (var i = 0; i < helpText.length; i++) {
    (function() {
       var item = helpText[i];
       document.getElementById(item.id).onfocus = function() {
         showHelp(item.help);
       }
    })(); // Immediate event listener attachment with the current value of item (preserved until iteration).
  }
}
```