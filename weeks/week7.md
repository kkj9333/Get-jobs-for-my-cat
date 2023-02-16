# Day 7 C++20协程和promise规范
## **前言**
前几天遇到一个问题，关于js单线程如何实现异步操作，我先入为主任务是通过多线程实现的。直达查阅了相关资料后才发现，由于回调地狱等问题，js在ES6以后的版本采用了协程配合promise生成器的方法来实现单线程异步调用。C++支持协程的进程很慢，C++20才加入了协程的一些东西，但是要实现封装需要到C++23才能在程序员中广泛使用，我之前虽然了解过协程的概念，但是一直以为这种技术很新（C++没有完全实装，java好像近几年才推广，大学教程压根没提过，好吧其实归根结底都是我比较无知没怎么了解过。实际上协程概念甚至先于线程），没想到其他语言已经用上了，今天来好好记录一下这个东西。
## **什么是协程**
协程就是一种特殊的函数，它可以在函数执行到某个地方的时候暂停执行，返回给调用者或恢复者（可以有一个返回值），并允许随后从暂停的地方恢复继续执行。**注意，这个暂停执行不是指将函数所在的线程暂停执行，而是单纯的暂停执行函数本身。**<br>
从cpu执行角度来讲，我理解为cpu中的一个线程利用执行间隙来执行协程函数，这样就提高了cpu的利用效率。<br>
在C++20中，co_return，co_yield，co_await是为了使用协程而新增加的三个关键字，这些关键字在非协程函数中是无法使用的。这也就意味着，在main函数中直接调用co_await xxxx(); 是不行的。这样听起来很费解，如果协程的关键字只能在协程函数中使用，而如何定义协程函数呢？难道只能递归吗？在这之前我们还需要了解一些概念。
## **协程帧**
当 caller 调用一个协程的时候会先创建一个协程帧，协程帧会构建 promise 对象，再通过 promise 对象产生 return object。协程帧中主要有这些内容：
- 协程参数
- 局部变量
- promise对象
这些内容在协程恢复运行的时候需要用到，**caller 通过协程帧的句柄 std::coroutine_handle 来访问协程帧。**
## **promise规范**
简单来说，Promise规范就是：如果在类A中定义一个叫做promise_type的结构体，并且其中包含特定名字的函数，那么这个类A就符合Promise规范，它就是一个符合Promise规范的类，它也就是一个Promise。
### promise_type
promise_type 是 promise对象的类型。promise_type 用于定义一类协程的行为，包括：
- 协程创建方式
- 协程初始化完成和结束时的行为
- 发生异常时的行为
- 如何生成 awaiter 的行为以及 co_return 的行为等等。
### coroutine return object
它是promise.get_return_object()方法创建的，一种常见的实现手法会将 std::coroutine_handle 存储到 coroutine object 内，使得该 return object 获得访问协程的能力。
### std::coroutine_handle
协程帧的句柄，主要用于访问底层的协程帧、恢复协程和释放协程帧。程序员可通过调用 std::coroutine_handle::resume() 唤醒协程。
### co_await、awaiter、awaitable
co_await：一元操作符；
awaitable：支持 co_await 操作符的类型；
awaiter：定义了 await_ready、await_suspend 和 await_resume 方法的类型。
co_await expr 通常用于表示等待一个任务(可能是 lazy 的，也可能不是)完成。co_await expr 时，expr 的类型需要是一个 awaitable，而该 co_await表达式的具体语义取决于根据该 awaitable 生成的 awaiter。

## **promise举例**
看起来和协程相关的对象还不少，这正是协程复杂又灵活的地方，可以借助这些对象来实现对协程的完全控制，实现任何想法。但是，需要先要了解这些对象是如何协作的，把这个搞清楚了，协程的原理就掌握了，写协程应用也会游刃有余了。
我们先来介绍一下promise 对象，它可以用于记录/存储一个协程实例的状态。每个协程桢与每个 promise 对象以及每个协程实例是一一对应的。
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
可以看到类task中定义了promise_type，同时其中包含了符合规范的5个函数，它就是一个Promise。
于是如果我们有以下定义：
```C++
task getTask() {
	// 实现中不需要返回task，也不能写return
	co_return;
}
```
所以根据协程规范，返回这个类的函数就是协程函数，task就是一个协程函数。
然而，这个时候task并不能使用co_await，我们之前已经了解到co_await是一个一元操作符，其后面只能跟随“一个实现了await_ready、await_suspend 和 await_resume三个特定函数的类”，即awaiter，所以我们还要实现这样一个类。下面是一个从网上找到的例子：
```C++
struct suspend_always_awaiter {
	constexpr bool await_ready() const noexcept { return false; }
	constexpr void await_suspend(std::coroutine_handle<> h) const noexcept {}
	constexpr void await_resume() const noexcept {}
};
```
当使用co_await suspend_always_awaiter(); 会发生什么呢？<br>
**首先会构造suspend_always_awaiter**，调用其构造函数（一般情况下我们就可以通过构造函数模仿一个普通的函数调用了）。<br>
**然后通过await_ready()判断是否需要等待**，如果返回true，就表示不需要等待，则立刻执行await_resume()；如果返回false，就表示需要等待，先执行await_suspend()，然后进入等待。_调用co_await awaitable(); 的函数会在这里暂停运行，但是不会影响所在线程的执行。_<br>
**await_suspend()函数中，我们可以通过传统的回调函数法执行一些异步操作**，然后在回调函数中调用std::coroutine_handle<>的resume()函数主动恢复。<br>
**await_resume()会在恢复执行后立刻执行**，_注意：co_wait的返回值就是该函数的返回值，而await_resume()函数允许拥有任意的返回值类型，模板类型也是允许的_。**也就是说可以用如下代码让co_wait的返回值更加自由**。<br>
```C++
template <class T>
struct someAsyncOpt {
	bool await_ready()
	void await_suspend(std::coroutine_handle<>);
	T await_resume();
};
```
我们以一个简单的代码来演示这些协程对象如何协作，并在下个部分介绍协作方式：
```C++
task foo () { 
    auto res = co_await suspend_always_awaiter; 
    co_return res ; 
}
```
## **协程驱动流程**
如下图就是基本执行流程图<br>
![image](https://user-images.githubusercontent.com/51207072/219366237-92d0c0a1-73d7-4a49-aee5-b99f2e62823e.png)<br>
浅蓝色部分的方法就是 Return_t 关联的 promise 对象的函数，浅红色部分就是 co_await 等待的 awaiter。


## **基本使用方式**
```javascript
clang-format [options] [<file> ...]
clang-format --help //建议至少浏览一遍帮助信息。
```
