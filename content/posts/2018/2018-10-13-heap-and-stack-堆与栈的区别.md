---
title: Heap And Stack 堆与栈的区别
author: admin
type: post
date: 2018-10-13T07:35:07+00:00
url: /archives/18398
categories:
 - 程序开发
 - 系统架构
tags:
 - 数据结构

---
**堆与栈的区别**

推荐： [https://www.programmerinterview.com/index.php/data-structures/difference-between-stack-and-heap/](https://www.programmerinterview.com/index.php/data-structures/difference-between-stack-and-heap/)

栈是上下顺序存储的，且“先进后出LIFO”规则，只能删除顶部的元素，而堆是没有特定的顺序的存储，您可以删除任意元素。堆分配需要维护分配的内存和未分配的内存的完整记录，以及一些开销维护以减少碎片，找到足够大以适应请求大小的连续内存段，等等。内存可以随时释放，留出自由空间。有时，内存分配器将执行维护任务，比如通过将分配的内存到处移动来对内存进行碎片整理，或者在运行时进行垃圾收集——当内存不再处于作用域中时对其进行标识并释放它。

1. 栈用在线程中，程序执行时由线程创建有限数量的栈空间，当线程结束的时候会自动回收，属于系统级。
堆一般是由应用程序在启动时创建，由应用程序回收，属于应用级。

> The stack is attached to a thread, so when the thread exits the stack is reclaimed. The heap is typically allocated at application startup by the runtime, and is reclaimed when the application (technically process) exits.

2. 栈的大小固定，通常在程序启动时已经确定了最大大小(部分语言支持动态扩容)。如果超出会出现“**_stack overflow”_**异常，经常在“递归”函数里出现。
堆是由”动态分配“的，其大小不固定，你可以在任何时间创建，再在任何时间回收（还是受限限系统虚拟内存 (ie: RAM and swap space)）。

3. 每个线程可有一个堆栈，而应用程序通常只有一个堆（尽管对于不同类型的分配有多个堆并不罕见）

4. 栈的读取速度要更快。因为分配内存的机制，只是移动指针，而堆还要做查找等操作。

> ## Which is faster – the stack or the heap? And why?
>
> The stack is much faster than the heap. This is because of the way that memory is allocated on the stack. Allocating memory on the stack is as simple as moving the stack pointer up.



> Variables allocated on the stack, or automatic variables, are stored directly to this memory. Access to this memory is very fast, and it’s allocation is dealt with when the program is compiled. Large chunks of memory, such as very large arrays, should not be allocated on the stack to avoid overfilling the stack memory (known as stack overflow). Stack variables only exist in the block of code in which they were declared. For example:

堆和栈在内存分配方面的不同： [http://timmurphy.org/2010/08/11/the-difference-between-stack-and-heap-memory-allocation/](http://timmurphy.org/2010/08/11/the-difference-between-stack-and-heap-memory-allocation/)

5. 堆和栈在用在函数变量的时候有些不同，一般动态创建的变量是存储在堆heap中，访问速度慢一些，而放在栈里的变量访问速度要快的多。

> Variables allocated on the heap, or dynamic variables, have their memory allocated at run time (ie: as the program is executing). Accessing this memory is a bit slower, but the heap size is only limited by the size of virtual memory (ie: RAM and swap space). This memory remains allocated until explicitly freed by the program and, as a result, may be accessed outside of the block in which it was allocated.

如编码中的定义一个变量 int i = 20 则变量 i 存储在stack中;创建一个对象 user = new Account(); //这时user变量则存储在heap中。

**堆与栈用法示例**

```
int foo()
{
  char *pBuffer; //<--nothing allocated yet (excluding the pointer itself, which is allocated here on the stack).
  bool b = true; // Allocated on the stack.
  if(b)
  {
    //Create 500 bytes on the stack
    char buffer[500];

    //Create 500 bytes on the heap
    pBuffer = new char[500];

   }//<-- buffer is deallocated here, pBuffer is not
}//<--- oops there's a memory leak, I should have called delete[] pBuffer;

```

[https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap](https://stackoverflow.com/questions/79923/what-and-where-are-the-stack-and-heap) [http://www.programmerinterview.com/index.php/data-structures/difference-between-stack-and-heap/](http://www.programmerinterview.com/index.php/data-structures/difference-between-stack-and-heap/)

视频 [https://www.youtube.com/watch?v=450maTzSIvA](https://www.youtube.com/watch?v=450maTzSIvA) [https://blog.csdn.net/waidazhengzhao/article/details/76651923](https://blog.csdn.net/waidazhengzhao/article/details/76651923)