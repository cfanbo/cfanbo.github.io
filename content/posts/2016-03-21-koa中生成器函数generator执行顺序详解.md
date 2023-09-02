---
title: koa中生成器函数generator执行顺序详解
author: admin
type: post
date: 2016-03-21T11:49:57+00:00
url: /archives/16822
categories:
 - 程序开发
tags:
 - koa

---

**ES6的generator** [http://book.apebook.org/minghe/koa-action/co/start.html](http://book.apebook.org/minghe/koa-action/co/start.html)

```

function* gen() {
  var a = yield 'start';
  console.log(a);
  var b = yield 'end';
  console.log(b);
  return 'over';
}
var it = gen();
console.log(it.next()); // {value: 'start', done: false}
console.log(it.next(22)); // 22 {value: 'end', done: false}
console.log(it.next(333)); // 333 {value: 'over', done: true}

```

带有 `*` 的函数声明表示是一个 generator 函数，当执行 `gen()` 时，函数体内的代码并没有执行，而是返回了一个 generator 对象。

generator 函数通常和 yield 结合使用，函数执行到每个 yield 时都会暂停并返回 yield 的右值。下次调用 next 时，函数会从 yield 的下一个语句继续执行。等到整个函数执行完，next 方法返回的 done 字段会变成 true，并且将函数返回值作为 value 字段。

第一次执行 `next()` 时，走到 `yield 'start'` 后暂停并返回 `yield` 的右值 `'start'`。注意，此时`var a =` 这个赋值语句其实还没有执行。

第二次执行 `next(22)` 时，从 `yield 'start'` 下一个语句执行。于是执行 `var a =` 这个赋值语句，而表达式 `yield 'start'` 的值就等于传递给 `next` 函数的参数值 `22`，所以，`a` 被赋值为 `22`。然后继续往下执行到 `yield 'end'` 后暂停并返回 `yield` 的右值 `'end'`。

第三次执行 `next(333)` 时，从 `yield 'end'` 下一个语句执行。此时执行 `var b =` 这个赋值语句，表达式 `yield 'end'` 的值等于传递给 `next` 函数的参数 `333`，`b` 被赋值为 `333`。继续往下执行到 `return` 语句，将 `return` 语句的返回值作为 `value` 返回，因为函数已经执行完毕，`done` 字段标记为 `true`。

可以看到 generator 就是一种迭代机制，就像一只很懒的青蛙，戳一下（调用 `next`）动一下。
出处： [http://taobaofed.org/blog/2016/03/18/error-handling-in-koa/](http://taobaofed.org/blog/2016/03/18/error-handling-in-koa/) [![2039474406-56efe8fb3a850_articlex](http://blog.haohtml.com/wp-content/uploads/2016/03/2039474406-56efe8fb3a850_articlex.gif)](http://blog.haohtml.com/wp-content/uploads/2016/03/2039474406-56efe8fb3a850_articlex.gif)