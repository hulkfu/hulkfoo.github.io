---
layout: post
title: node.js中的exports
---

exports在node.js中用来进行工程的管理的。

如果不用exports，那么所以类、方法都写在一个文件里。这样即乱，也不能很好的实现接口。

exports就是把代码分开再合起来。


# exports vs module.exports
exports是对module.exports的省略写法，用exports的地方完全可以用后者代替。但module.exports可以直接等于一个函数，exports就不能。

```
// car.js

var Car = function() {
	// ...对类的实现
}

module.exports = Car;
```

这样便可以在其他文件中使用这个“类”了：

```
var Car = require('car');

var car1 = new Car();
```

exports只能通过后面加点然后跟函数名来导出。如：

```
// car.js

var go = function() {}

exports.go = go
```

然后使用时也只能通过变量用导出的方法：

```
var car = require('car')
car.go()
```

通过使用module.exports出一个变量，可以实现命名空间的功能；而一般的exports是exports出实现方法。

