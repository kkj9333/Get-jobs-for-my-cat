# Day 7 C++20协程和promise规范
## **前言**
前几天遇到一个问题，关于js单线程如何实现异步操作，我先入为主任务是通过多线程实现的。直达查阅了相关资料后才发现，由于回调地狱等问题，js在ES6以后的版本采用了协程配合promise生成器的方法来实现单线程异步调用。C++支持协程的进程很慢，C++20才加入了协程的一些东西，但是要实现封装需要到C++23才能在程序员中广泛使用，我之前虽然了解过协程的概念，但是一直以为这种技术很新（因为C++没有，java好像近几年才推广，实际上协程概念甚至先于线程，但是大学教程压根没提过这玩意），没想到其他语言已经用上了，今天来好好记录一下这个东西。
## **什么是协程**
协程就是一种特殊的函数，它可以在函数执行到某个地方的时候暂停执行，返回给调用者或恢复者（可以有一个返回值），并允许随后从暂停的地方恢复继续执行。**注意，这个暂停执行不是指将函数所在的线程暂停执行，而是单纯的暂停执行函数本身。**<br>
在C++20中，co_return，co_yield，co_await是为了使用协程而新增加的三个关键字，这些关键字在非协程函数中是无法使用的。这也就意味着，在main函数中直接调用co_await xxxx(); 是不行的。这样听起来很费解，如果协程的关键字只能在协程函数中使用，而如何定义协程函数呢？难道只能递归吗？在这之前我们还需要了解一些概念。
## **协程帧**
当 caller 调用一个协程的时候会先创建一个协程帧，协程帧会构建 promise 对象，再通过 promise 对象产生 return object。协程帧中主要有这些内容：
- 协程参数
- 局部变量
- promise对象
这些内容在协程恢复运行的时候需要用到，caller 通过协程帧的句柄 std::coroutine_handle 来访问协程帧。
## **promise规范**
简单来说，Promise规范就是：如果在类A中定义一个叫做promise_type的结构体，并且其中包含特定名字的函数，那么这个类A就符合Promise规范，它就是一个符合Promise规范的类，它也就是一个Promise。
promise_type 是 promise对象的类型。promise_type 用于定义一类协程的行为，包括：
- 协程创建方式
- 协程初始化完成和结束时的行为
- 发生异常时的行为
- 如何生成 awaiter 的行为以及 co_return 的行为等等。
<br>promise 对象可以用于记录/存储一个协程实例的状态。每个协程桢与每个 promise 对象以及每个协程实例是一一对应的。

如下是我从网上找到的一个例子：<br>
```C++
struct task{
  struct promise_type {
    auto get_return_object() { return task{}; }
    std::suspend_never initial_suspend() { return {}; }
    std::suspend_never final_suspend() { return {}; }
    void return_void() {}
    void unhandled_exception() {}
  };
}
```

## **基本使用方式**
```javascript
clang-format [options] [<file> ...]
clang-format --help //建议至少浏览一遍帮助信息。
```
