---
layout: post
title: angularjs
---

directives

* ng-model
* ng-click
* ng-repeat
* ng-show/ng-hide
* ng-init

在directive里引用ng-model等变量，就直接用，不要加{{}}。因为它们都是在一个域里了，加了反而什么都找不到。

# $scope

View和Controller都显性的声明了，而Model就是scope了。在scope里面定义数据和方法，反正对js来说都是key-value。

那么这里为什么不用model呢？因为scope里可能会有不只一个model。

每一次嵌套使用controller，它都是在上一层的原型基础上，最上的是$rootScope。

# module
用来定义模块。app，controller，service及任何想要定义的模块。

对，它就是用来组织代码的模块。

但在定义时需要用angular的语法：

```
angular.module('module', ['dependentModule1', 'dependentModule2'])
.config() {}
.controller('XxController') {['$http', function($http) {    // 这叫dependency injection，要使用的服务要先声明一下，之后准备好后才会执行后面的function
}]}
.factory() {}
```

## Dependency Injection
https://docs.angularjs.org/guide/di

虽然依赖能够自动注入，但是当你minify你的js代码后，参数名字会变，可是字符串不变。

Careful: If you plan to minify your code, your service names will get renamed and break your app.

所以Angular是定义了一种新的语言，而且它还能自己定义关键字，即directive。通过这些directives，将js与html穿了起来。在html里，directive就是标签的属性。

# controller
定义的contreoller是待传给其的参数都准备好后自动调用的，参数像是其依赖的模块。

喜欢angular是因为它把逻辑做的很清晰，有一种归一的感觉。

# $watch

# service

# route

# filter

# 插件

## ui-route

state是根据url来判断当前的状态的，然后state里controller的参数命名也是url定义的。

```
  .state('tab.book-detail', {
      url: '/books/:bookId',
      views: {
        'tab-books': {
          templateUrl: 'templates/book-detail.html',
          controller: 'BookDetailCtrl'
        }
      }
    })
```

# 参考
* http://www.ng-newsletter.com/posts/beginner2expert-how_to_start.html
* https://docs.angularjs.org/tutorial