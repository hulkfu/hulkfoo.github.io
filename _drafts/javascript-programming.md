---
layout: post
title : JavaScript编程
---

JavaScript是一门简单但完备的语言。

# prototype
正是prototype体现了JavaScript的Script，其它的都是Java了。
通过prototype，可以实现类、继承进而其它高级的设计模式。

## 构造函数
和一般函数一样，只是为了便于认出首字母大写，然后通过**new**来创建新实例。

```javascript
function Persion(name) {
  this.name = name;
  this.who = function() {
    console.log(this.name);
  }
}

var p1 = new Persion('Jack');
var p2 = new Persion('Tom');
p1.who()  // #=> Jack
```

在使用**new**来创建新实例时，会经历下面4个步骤：

* 创建一个新的对象。
* 将构造函数的作用域赋给新对象（因此this就指向了这个新对象）。
* 执行构造函数中的代码（为这个新对象添加属性）。
* 返回新对象。

由于在JavaScript中函数也是对象，因此上例中的p1和p2的who方法是不同的。也就是说每次实例化一个对象，都会重新定义一遍方法，显然这是不可取的。
因此会选择在构造函数的prototype中定义实例的方法以及实例间共享的变量。

## 寄生组合式继承

# 闭包
通过闭包，JavaScript实现了命名空间的隔离。


