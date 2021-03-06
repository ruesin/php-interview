# 协程

Coroutine（协程）是一种用户态的轻量级线程，又称微线程。

协程不是进程或线程，其执行过程更类似于子例程，或者说不带返回值的函数调用。子程序就是协程的一种特例。

协程就是一个函数，一个特殊的函数——可以在某个地方挂起，并且可以重新在挂起处继续运行。

相对子例程而言，协程更为一般和灵活，但在实践中使用没有子例程那样广泛。

子程序，或者称为函数，在所有语言中都是层级调用，比如A调用B，B在执行过程中又调用了C，C执行完毕返回，B执行完毕返回，最后是A执行完毕。

所以子程序调用是通过栈实现的，一个线程就是执行一个子程序。子程序调用总是一个入口，一次返回，调用顺序是明确的。

而协程的调用和子程序不同，执行过程中，在子程序内部可中断，然后转而执行别的子程序，在适当的时候再返回来接着执行。注意，在一个子程序中中断，去执行其他子程序，不是函数调用，有点类似CPU的中断。

一个进程可以包含多个线程，一个线程也可以包含多个协程，也就是说，一个线程内可以有多个那样的特殊函数在运行。但是，一个线程内的多个协程的运行是串行的。如果有多核CPU的话，多个进程或一个进程内的多个线程是可以并行运行的，但是一个线程内的多个协程却绝对串行的，无论有多少个CPU（核）。协程虽然是一个特殊的函数，但仍然是一个函数。一个线程内可以运行多个函数，但是这些函数都是串行运行的。当一个协程运行时，其他协程必须挂起。

一个程序可以包含多个协程，可以对比与一个进程包含多个线程。多个线程相对独立，有自己的上下文，切换受系统控制；而协程也相对独立，有自己的上下文，但是其切换由自己控制，由当前协程切换到其他协程由当前协程来控制。

协程的特点在于是一个线程执行，和多线程比，最大的优势就是协程极高的执行效率，因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显；第二个优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，可以通过多进程+协程的方式。既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

并行(parallel)：指在同一时刻，有多条指令在多个处理器上同时执行。（多进程）

并发(concurrency)：指在同一时刻只能有一条指令执行，但多个进程指令被快速的轮换执行，使得在宏观上具有多个进程同时执行的效果，但在微观上并不是同时执行的，只是把时间分成若干段，使多个进程快速交替的执行。（协程）

网络爬虫主要受到网络延迟和本地运行效率的限制，传统的基于多线程的网络爬虫架构主要为了消除网络延迟而没有考虑到本地运行效率。在高并发的条件下，多线程架构爬虫由于上下文切换开销增大而导致本地运行效率降低，同时使得网络利用率下降，单进程内网络IO阻塞导致CPU、IO无法充分利用。

## 4.0 协程实现原理
### 内存栈
4.0版本使用了PHP+C的双栈模式。创建协程时会创建一个C栈，默认尺寸为2M，创建一个PHP栈，默认为8K。

C栈主要用于保存底层函数调用的局部变量数据，用于解决call_user_func、array_map等C函数调用在协程切换时未能还原的问题。4.0版本无论如何切换协程，底层总是能正确地切换回原先的C函数栈帧继续向下执行。

C栈分配的2M内存，使用了虚拟内存，并不会分配实际内存

PHP栈主要保存PHP函数调用的局部变量数据，主要是zval结构体，PHP中标量类型，如整型、浮点型、布尔型等是直接保存在zval结构体内的，而object、string、array是使用引用计数管理，在堆上存储的。8K的PHP栈足以保存整个函数调用的局部变量。

### 协程切换
C栈切换使用了boost.context 1.60汇编代码，用于保存寄存器，切换指令序列。主要是jump_fcontext这个ASM函数提供。PHP栈的切换是跟随C栈切同步进行的。底层会切换EG(vm_stack)使得PHP恢复到正确的PHP函数栈帧。4.0.2版本还增加了ob输出缓存区的切换，ob_start等操作也可以用于协程。

boost.context汇编切换协程栈的效率非常高，经过测试每秒可完成2亿次切换

某些平台下不支持boost.context汇编，底层将使用ucontext

### 协程调度
4.0协程实现中，主协程即为Reactor协程，负责整个EventLoop的运行。主协程实现事件监听，在IO事件完成后唤醒其他工作协程。

#### 协程挂起
在工作协程中执行一些IO操作时，底层会将IO事件注册到EventLoop，并让出执行权。

- 嵌套创建的非初代协程，会逐个让出到父协程，直到回到主协程
- 在主协程上创建的初代协程，会立即回到主协程
- 主协程的Reactor会继续处理IO事件、Wait监听新事件（epoll_wait）

初代协程是在EventLoop内直接创建的协程，例如OnRecive回调方法中的内置协程就是初代协程

#### 协程恢复
当主协程的Reactor接收到新的IO事件，底层会挂起主协程，并恢复IO事件对应的工作协程。该工作协程挂起或退出时，会再次回到主协程。

## 协程执行流程
协程执行流程遵循以下原则:
- 协程没有IO等待 正常执行PHP代码，不会产生执行流程切换
- 协程遇到IO等待 立即将控制权切，待IO完成后，重新将执行流切回原来协程切出的点
- 协程并行协程依次执行，同上一个逻辑
- 协程嵌套执行流程由外向内逐层进入，直到发生IO，然后切到外层协程，父协程不会等待子协程结束

## Swoole4 协程与 Go 协程
Swoole4与Go协程在设计上是完全一致的，均是stackful的，每个协程拥有独立的运行栈。协程调度器使用汇编代码，切换协程上下文。

### 多线程
Swoole4的协程调度器是单线程的，因此不存在数据同步问题，同一时间只会有一个协程在运行。

Go协程调度器是多线程的，同一时间可能会有多个协程同时执行。

因此在Swoole4协程中操作全局变量是不需要加锁的。而Go的程序由于依然是类似Java的多线程模式，因此务必要对临界资源加锁，避免出现数据同步问题。或者使用官方sync包提供的各种并发容器。

实际上Go的chan和并发容器，底层仍然使用了Mutex进行锁操作，锁的争抢是普遍存在的。

Swoole4由于是单线程多进程的，底层没有使用任何Mutex锁，不存在锁的争抢。同样带来的问题是，没有超全局变量。只有进程级全局变量，读写PHP全局变量只在当前进程内有效。

如果希望多进程共享数据，有3种解决方案：
- 使用Table和Atomic对象，或者其他共享内存数据结构
- 使用IPC进程间通信
- 借助存储实现数据的共享和中转，如Redis、MySQL或文件操作

### Defer
Go的defer是绑定函数的，在当前函数退出时会执行defer任务。这是由于Go没有析构函数，可能会出现资源泄漏。而PHP是有析构函数的，所有资源类对象，析构时会自动释放。

Swoole4的defer设计为在协程退出时一起执行，在多层函数调用嵌套中添加大量defer任务，与Coroutine是绑定的。在这个Coroutine结束时，会按照先进后出的顺序，执行defer任务。

### 共用 Socket 资源
在Go的程序中，可以多个协程同时并发地去读同一个Socket，底层完成了协程的排队和调度。

这要求开发者清楚自己的行为，否则可能会产生逻辑错误。
```go
func test() {
    db := new(database)
    close := db.connect()

    go func(db) {
        db.query(sql);
    } (db);

    go func(db) {
        db.query(sql);
    } (db);
}
```
Go是允许这样操作的，实际上这个可能会存在严重问题。socket读写操作产生并发，可能产生数据包错乱。

Swoole中禁止了这种行为。不允许多个协程同时读取同一个Socket。否则会产生致命错误。

```php
function test() {
    $db = new Database;
    $db->connect('127.0.0.1', 6379);

    go(function () use ($db) {
        $db->query($sql);
    });

    go(function () use ($db) {
        $db->query($sql);
    });
}
```
以上代码中有2个协程同时操作$db对象，可能会产生严重错误。底层会直接抛出致命错误，错误信息为：
```
"%s has already been bound to another coroutine#%ld, 
reading or writing of the same socket in multiple coroutines at the same time is not allowed."
```

错误码：SW_ERROR_CO_HAS_BEEN_BOUND

使用Channel或SplQueue实现连接池，管理资源对象，就可以很好地解决此问题。

参考：
- https://wiki.swoole.com/wiki/page/p-coroutine_realization.html