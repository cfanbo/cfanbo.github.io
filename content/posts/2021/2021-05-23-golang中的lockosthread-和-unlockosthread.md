---
title: Golang中的runtime.LockOSThread 和 runtime.UnlockOSThread
author: admin
type: post
date: 2021-05-23T15:06:06+00:00
url: /archives/30780
toc: true
categories:
 - 程序开发
tags:
 - golang

---
在runtime中有 ` [runtime.LockOSThread](https://github.com/golang/go/blob/go1.16.3/src/runtime/proc.go#L4248-L4278) ` 和 ` [runtime.UnlockOSThread](https://github.com/golang/go/blob/go1.16.3/src/runtime/proc.go#L4302-L4323) ` 两个函数，这两个函数有什么作用呢？我们看一下标准库中对它们的解释。

## runtime.LockOSThread 

```
// LockOSThread wires the calling goroutine to its current operating system thread.
// The calling goroutine will always execute in that thread,
// and no other goroutine will execute in it,
// until the calling goroutine has made as many calls to
// UnlockOSThread as to LockOSThread.
// If the calling goroutine exits without unlocking the thread,
// the thread will be terminated.
//
// All init functions are run on the startup thread. Calling LockOSThread
// from an init function will cause the main function to be invoked on
// that thread.
//
// A goroutine should call LockOSThread before calling OS services or
// non-Go library functions that depend on per-thread state.
```

调用 `LockOSThread` 将 `绑定` 当前 `goroutine` 到当前 操作系统 `线程`，此 `goroutine` 将始终在此线程执行，其它 `goroutine` 则无法在此线程中得到执行，直到当前调用线程执行了 `UnlockOSThread` 为止（也就是说 `LockOSThread` 可以指定一个goroutine `独占` 一个系统线程）；

如果调用者goroutine 在未解锁线程（未调用 `UnlockOSThread`）之前直接退出，则当前线程将直接被终止（也就是说线程被直接销毁）。参考《 [Golang LockOSThread子协程绑定问题](http://xiaorui.cc/archives/5320)》

我们知道在golang中创建的线程正常情况下是无法被销毁的，但通过这种hack用法则可以将其销毁，参考《 [极端情况下收缩 Go 的线程数](https://xargin.com/shrink-go-threads/)》。

所有 `init函数` 都运行在启动线程。如果在一个 `init函数` 中调用了 `LockOSThread` 则导致 `main` 函数被执行在当前线程。

goroutine应该在调用依赖于每个线程状态的 `OS服`务 或 `非Go库函数` 之前调用 `LockOSThread`。

**总结**

`LockOSThread` 是用来将 goroutine 与系统线程的关系进行绑定，起到 `独占系统线程` 的作用，带来的影响是会导致其它 goroutine 无法在当前系统线程中得到执行（`goroutine的调度机制`），除非调用 `UnlockOSThread` 函数。

如果在在未调用 `UnlockOSThread` 前直接退出的话，则此线程将直接销毁。

**嵌套调用**

另外还有一种情况是 嵌套调用 `LockOSThread` 和 `UnlockOSThred` 的情况。即当前 `goroutine` 里调用了`LockOSThread`, 同时又创建了一个新的`goroutine`的情况，官方对此进行了解释 [https://tip.golang.org/doc/go1.10#runtime](https://tip.golang.org/doc/go1.10#runtime) 。子协程是不会继承父协程的 `LockOSThread` 特性的。

## runtime.UnlockOSThread 

```
// UnlockOSThread undoes an earlier call to LockOSThread.
// If this drops the number of active LockOSThread calls on the
// calling goroutine to zero, it unwires the calling goroutine from
// its fixed operating system thread.
// If there are no active LockOSThread calls, this is a no-op.
//
// Before calling UnlockOSThread, the caller must ensure that the OS
// thread is suitable for running other goroutines. If the caller made
// any permanent changes to the state of the thread that would affect
// other goroutines, it should not call this function and thus leave
// the goroutine locked to the OS thread until the goroutine (and
// hence the thread) exits.
```

`UnlockOSThread` 用于取消对 `LockOSThread` 的调用。

如果将调用 `goroutine` 的活动 `LockOSThread` 调用数减为零，它从其固定的操作系统线程中取消调用 `goroutine`。即如果多次调用 `LockOSThread`，则必须将 `UnlockOSThread` 调用相同的次数才能解锁线程。

如果没有活动的 `LockOSThread` 调用，则这是一个空操作。

在调用 `UnlockOSThread` 之前，调用者要确保当前线程适合运行其它goroutine。如果调用者对线程的状态做了任何永久性的更改，这将影响其他goroutine，那么它不应该调用此函数，从而使goroutine锁定到OS线程，直到goroutine（以及线程）退出，此时系统线程直接被销毁。

**总结**

`UnlockOSThread` 是对 `LockOSThread` 的反向操作，即解锁。用户可能在 `LockOSThread` 期间对操作系统线程进行了修改，所以`有可能`不再适合其它goroutine在当前系统线程运行，这种情况下就不再需要调用 `UnlockOSThread`，而是直接等待goroutine执行完毕自动退出。

## 参考资料 

 * [https://tip.golang.org/doc/go1.10#runtime](https://tip.golang.org/doc/go1.10#runtime)
 * [https://stackoverflow.com/questions/25361831/benefits-of-runtime-lockosthread-in-golang](https://stackoverflow.com/questions/25361831/benefits-of-runtime-lockosthread-in-golang)
 * [http://xiaorui.cc/archives/5320](http://xiaorui.cc/archives/5320)
 * [极端情况下收缩 Go 的线程数](https://xargin.com/shrink-go-threads/)