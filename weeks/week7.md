# Day 7 C++20协程和promise规范
## **前言**
前几天遇到一个问题，关于js单线程如何实现异步操作，我先入为主任务是通过多线程实现的。直到查阅了相关资料后才发现，由于回调地狱等问题，js在ES6以后的版本采用了“协程”（循环事件模拟的）配合promise生成器的方法来实现单线程异步调用。C++支持协程的进程很慢，C++20才加入了协程的一些东西，但是要实现封装需要到C++23才能在程序员中广泛使用，我之前虽然了解过协程的概念，但是一直以为这种技术很新（C++没有完全实装，java好像近几年才推广，大学教程压根没提过，好吧其实归根结底都是我比较无知没怎么了解过。实际上协程概念甚至先于线程），没想到其他语言已经用上了，今天来好好记录一下这个东西。
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
### co_await机制
co_await 操作符是 C++20 新增的一个关键字，**co_await expr 一般表示等待一个惰性求值的任务**，这个任务可能在某个线程执行，也可能在 OS 内核执行，什么时候执行结束不知道，为了性能，我们又不希望阻塞等待这个任务完成，所以就借助 co_await 把协程挂起并返回到 caller，caller 可以继续做事情，当任务完成之后协程恢复并拿到 co_await 返回的结果。
所以 co_await 一般有这几个作用：
- 挂起协程；
- 返回到 caller；
- 等待某个任务(可能是 lazy 的，也可能是非 lazy 的)完成之后返回任务的结果。

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
浅蓝色部分的方法就是 Return_t 关联的 promise 对象的函数，浅红色部分就是 co_await 等待的 awaiter。<br>
首先需要创建协程，创建协程的流程大概如下：<br>
- 创建一个协程帧（coroutine frame）；
- 在协程帧里构建 promise 对象；
- 把协程的参数拷贝到协程帧里；
- 调用 promise.get_return_object() 返回给 caller 一个对象，即代码中的 Return_t 对象。
创建协程之后是否挂起则由调用者设置 initial_suspend() 的返回类型来确定。<br>
在这个模板框架里有一些可定制点：如 initial_suspend()、final_suspend()、unhandled_exception() 和 return_value()；又比如awaiter的await_ready()
我们可以通过 promise 的 initial_suspend() 和 final_suspend() 返回类型来控制协程是否挂起，在 unhandled_exception() 里处理异常，在 return_value() 里保存协程返回值。
可以根据需要定制 initial_suspend 和 final_suspend 的返回对象来决定是否需要挂起协程。如果挂起协程，代码的控制权就会返回到caller，否则继续执行协程函数体（function body）。<br>
_另外值得注意的是，如果禁用异常，那么生成的代码里就不会有 try-catch。此时协程的运行效率几乎等同非协程版的普通函数。这在嵌入式场景很重要，也是协程的设计目的之一。_<br>
通过定制 awaiter.await_ready() 的返回值就可以控制是否挂起协程还是继续执行，返回 false 就会挂起协程，并执行 awaiter.await_suspend()，通过 awaiter.await_suspend() 的返回值来决定是返回 caller 还是继续执行。<br>
_C++20协程中最重要的两个对象就是 promise 对象(恢复协程和获取某个任务的执行结果)和 awaiter(挂起协程，等待task执行完成)，其它的都是“工具人”，要实现想要的的协程，关键是要设计如何让这两个对象协作好_。更多细节可以看：
https://link.zhihu.com/?target=https%3A//lewissbaker.github.io/2017/11/17/understanding-operator-co-await%25EF%25BC%2589%25E3%2580%2582


## **完整举例**
这是一个从网上找的例子，用用协程在一个线程里打印线程id，这个输出可以清晰的看到协程是如何创建的、co_await 等待线程结束、线程结束后协程返回值以及协程销毁的整个过程。
```C++
#include <coroutine>
#include <iostream>
#include <thread>
 
namespace Coroutine {
  struct task {
    struct promise_type {
      promise_type() {
        std::cout << "1.create promie object\n";
      }
      task get_return_object() {
        std::cout << "2.create coroutine return object, and the coroutine is created now\n";
        return {std::coroutine_handle<task::promise_type>::from_promise(*this)};
      }
      std::suspend_never initial_suspend() {
        std::cout << "3.do you want to susupend the current coroutine?\n";
        std::cout << "4.don't suspend because return std::suspend_never, so continue to execute coroutine body\n";
        return {};
      }
      std::suspend_never final_suspend() noexcept {
        std::cout << "13.coroutine body finished, do you want to susupend the current coroutine?\n";
        std::cout << "14.don't suspend because return std::suspend_never, and the continue will be automatically destroyed, bye\n";
        return {};
      }
      void return_void() {
        std::cout << "12.coroutine don't return value, so return_void is called\n";
      }
      void unhandled_exception() {}
    };
 
    std::coroutine_handle<task::promise_type> handle_;
  };
 
  struct awaiter {
    bool await_ready() {
      std::cout << "6.do you want to suspend current coroutine?\n";
      std::cout << "7.yes, suspend becase awaiter.await_ready() return false\n";
      return false;
    }
    void await_suspend(
      std::coroutine_handle<task::promise_type> handle) {
      std::cout << "8.execute awaiter.await_suspend()\n";
      std::thread([handle]() mutable { handle(); std::cout << "?.tread has been executed,the thread id=" << std::this_thread::get_id() << "\n"; }).detach();
      std::cout << "9.a new thread lauched, and will return back to caller\n";
    }
    void await_resume() {
      std::cout << "?.await_resume is executed\n";
    }
  };
 
  task test() {
    std::cout << "5.begin to execute coroutine body, the thread id=" << std::this_thread::get_id() << "\n";//#1
    co_await awaiter{};
    std::cout << "11.coroutine resumed, continue execcute coroutine body now, the thread id=" << std::this_thread::get_id() << "\n";//#3
  }
}// namespace Coroutine
 
int main() {
  Coroutine::test();
  std::cout << "10.come back to caller becuase of co_await awaiter\n";
  std::this_thread::sleep_for(std::chrono::seconds(1));
  std::cout << "?.sleep over" << std::endl;
  return 0;
}
```
输出结果如下，对照之前的协程驱动流程图会更好理解些：
![image](https://user-images.githubusercontent.com/51207072/219549145-e3606b04-bf35-4a3a-8f30-f0fee128b49b.png)
<br>
这里其实调用挺奇怪的，这是因为使用了协程关键字的协程函数以及协程关键字会生成一些额外的代码来支持协程操作，这里task并非在返回时才会生成，而是通过get_return_object初始化器列表直接生成了，这也意味着如果你重写了task的默认构造函数，那这个初始化器列表的构造函数你也必须重写。
简单来梳理一下流程：<br>
首先是创建协程，先创建一个协程帧（coroutine frame），这里我们看不到；在协程帧里构建 promise对象，**1.创造promise_type即task::promise_type结构**；把协程的参数拷贝到协程帧里；
调用 promise.get_return_object()**2.创建协程返回对象即task并把std::coroutine_handle<task::promise_type>协程的句柄拷贝到task，到此协程就创建好了**。<br>
然后会调用promise.inital_suspend()来决定是否挂起协程，**3.是否挂起协程?4.不挂起因为返回 std::suspend_never,所以协程函数继续执行**。<br>
如果你是debug模式下，你会发现F11也只会断在这里，也就是说1-4的执行不中断。
![image](https://user-images.githubusercontent.com/51207072/219542754-bea9ac39-45db-4f73-8288-b73d2e830596.png)<br>
此时转到协程函数主体执行，并打印相应线程id，也就是main函数线程id，**5.执行协程函数主体，目前线程id是main函数的**。
![image](https://user-images.githubusercontent.com/51207072/219542839-c3f9c803-768c-4022-b62b-a29bb3d63ba0.png)<br>
继续执行，遇到协程关键字co_await，这时候会执行awaiter的await_ready()，**6.要暂停协程吗？7.是的因为await_ready()返回false**。<br>
然后这里我们会因为返回了false而执行awaiter的await_suspend(handle)，**8.执行awaiter.await_suspend()**。**9.新建线程，传入handle（handle可以主动地resume协程）并detach**，在suspend执行完后，（因为return void）会返回协程函数调用者（也即是main函数）继续执行：**10.因为co_await awaiter语句而继续执行caller的代码**：
![image](https://user-images.githubusercontent.com/51207072/219545231-e2bc3d58-8f65-40af-af01-849b541c0085.png)<br>
然后主线程休眠（似乎意思是会主动触发resume，**?.await_resume is executed**），**11.协程被唤醒, 现在继续执行协程函数, 目前线程id是新线程的**，这里有点难以理解，但是协程是利用cpu的工作间隙去执行的，可能是两个线程都分配在一个核里，所以是并发的，一个线程休眠时，另外一个线程被唤起，当新线程开始运行的时候恢复挂起的协程，handle()会主动resume协程，这时候代码执行会回到协程函数继续执行，但是此时获取的线程id是就是另一个线程的，这也就是说为什么协程是线程无关的。（_caller 这时候可以不用阻塞等待线程结束，可以做其它事情。更多时候我们在线程完成之后才去恢复协程，这样可以告诉挂起等待任务完成的协程：任务已经完成了，现在可以恢复了，协程恢复后拿到任务的结果继续执行。注意：这里的 awaiter 同时也是一个 awaitable，因为它支持 co_await_。）<br>
然后就是协程函数执行完毕时的返回值，**12.由于协程函数不返回任何值, 所以promise.return_void()被调用**。<br>
之后会进入promise.final_suspend() 决定是否要自动销毁协程，返回 std::suspend_never 就自动销毁协程，否则需要用户手动去销毁。
**13.协程函数执行完了，你要挂起这个协程吗**？
**14.因为返回std::suspend_never而不挂起，之后协程也会自动销毁**。<br>
到这里协程函数就执行完了，但是新线程还没有，于是继续执行**?.tread has been executed,the thread id=xxx**。线程执行完后销毁，直到主线程休眠完毕，然后执行
**?.sleep over**。这就是整个协程的执行流程。<br>
## 协程库
可以看出C++20的协程关键字真的很难理解，而且需要开发者考虑很多细枝末节的东西，如果直接使用可能会很不方便，这边收集了一些比较方便的协程库，可以参考一下：
- C++20 协程库 async_simple https://github.com/alibaba/async_simple
