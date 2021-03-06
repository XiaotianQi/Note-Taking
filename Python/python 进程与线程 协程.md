## 简介

程序开发的一大矛盾是，要用控制流去完成逻辑流。也就是说，要用指令的执行来完成逻辑链条的前因后果。

在刚开始学程序的时候，往往都是从控制流等价于执行流的情况下学起，执行到哪，就意味着逻辑走到了哪。这样的程序结构清晰，可读性好。

但是问题是中间有些过程是不能立即得到结果的，程序为了等结果就会阻塞。这种情况多见于一些io操作。为了提升效率，我们可以使用异步的api，通过回调/通知函数来响应操作结果，同时接着执行下一轮的逻辑。

异步回调/通知的问题在于，它把原本统一的逻辑流拆开成了几个阶段，这样控制流和逻辑流就不等价了。为了保证逻辑数据的传递，需要自己来维护状态，阅读起来也比较头疼。状态的维护历来就是bug层出的地方，很容易掉入无效状态而死掉。同时，这种机制调试起来还特别麻烦，因为状态所能提供的信息不够，往往还得手动跟踪调用链，这是相当费精力的。

协程是一种任务调度机制，它可以让你用逻辑流的顺序去写控制流，而且还不会导致操作系统级的线程阻塞。你发起异步请求、注册回调/通知器、保存状态，挂起控制流、收到回调/通知、恢复状态、恢复控制流的所有过程都能过一个yield来默默完成。

从代码结构上看，协程保证了编写过程中的思维连贯性，使得函数（闭包）体本身就无缝保持了程序状态。逻辑紧凑，可读性高，不易写出错的代码，可调试性强。

从实现上看，与线程相比，这种主动让出型的调度方式更为高效。一方面，它让调用者自己来决定什么时候让出，比操作系统的抢占式调度所需要的时间代价要小很多。后者为了能恢复现场会在切换线程时保存相当多的状态，并且会非常频繁地进行切换。另一方面，协程本身可以做在用户态，每个协程的体积比线程要小得多，因此一个进程可以容纳数量相当可观的逻辑流。

* 多线程阻塞的执行图如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%204.jpg)

多线程情形下的多逻辑能力严重依赖于程序申请到的执行流/线程的数量。线程是很重的，不仅体积庞大，还得经常进出内核。在这种模式下，线程的阻塞等待浪费了大量内存。

* 回调的执行图例如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%205.jpg)

异步回调可以避免忙等，但是逻辑流被拆开了，收到回调之后很难恢复出是从哪里调过来的，逻辑流栈上的变量也被清空了。

* 带有协程的执行图例如下：

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20asyncio%206.jpg)

协程不会出现多线程这种死等的情况，一但遇到阻塞，那么要么从池子里挑下一个准备好的协程跑，要么由阻塞的协程指定由谁接着它跑，没有CPU时间的浪费。同时相比于图2，逻辑流在程序员看来是连续的，所有的状态都通过原生的栈延续得好好的。

**协程只是一种任务调度模式，虽然很多情形下是用来解决I/O问题，但它不必然与实际的I/O相关。**在上面的方式二中我们可以看出，有时候需要唤醒的下一个逻辑流其实可以通过挂起的逻辑流来指定，这在生产者-消费者模式中非常常见：

生产者生产出产品->挂起生产者，通知消费者消费->消费者被唤醒，消费产品->挂起消费者，通知生产者生产-> ...

单线程的异步编程模型称为协程，单线程控制权交替。

现代操作系统对I/O操作的改进中最为重要的就是支持异步I/O。如果充分利用操作系统提供的异步I/O支持，就可以用单进程单线程模型来执行多任务。

清晰优雅的协程可以说实现异步的最优方案之一。**事件驱动**模型就是异步编程的重中之重。

异步框架就是建立在非阻塞和事件循环两个特征之上的, 这就在单线程中实现了并发操作。事件循环是高效并发I/O的，因为并没有为每一条连接都分配线程资源。在Python中，像我们所创建的事件循环，在处理数量不大但交互频繁的连接时，要比多线程方式慢。在没有全局解释器锁(Global Interpreter Lock)的运行时环境下，多线程同样表现得更出色。而异步I/O的真正用武之处在于，充满大量慢连接和不频繁事件的应用场景。

通常在Python中我们进行并发编程一般都是使用多线程或者多进程来实现的，对于CPU计算密集型任务由于GIL的存在通常使用多进程来实现，而对于IO密集型任务可以通过线程调度来让线程在执行IO任务时让出GIL，从而实现表面上的并发。

其实对于IO密集型任务我们还有一种选择就是协程。协程，又称微线程，英文名Coroutine。Python中的异步IO模块asyncio就是基本的协程模块。可以多次进出、每次暂停并恢复执行的函数被称为协程。

### 协程和线程、进程

- **进程**：操作系统资源分配的最小单位。 每个进程有独立的内存空间，互不干扰，创建进程需要创建或者拷贝进程空间，占用很多资源，进程切换需要内核切换上下文
- **线程**：操作系统能够进行运算调度的最小单位。它被包含在进程之中，是进程中的实际运作单位。同一个进程内，线程共享进程空间的任务单位，但有各自的调用栈（call stack），自己的寄存器环境（register context），自己的线程本地存储（thread-local storage）。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不独立拥有系统资源，但它可与同属一个进程的其它线程共享该进程所拥有的全部资源。每一个应用程序都至少有一个进程和一个线程。线程切换需要内核切换上下文。
- **协程**：是计算机程序的一类组件，推广了非抢先多任务的子程序，允许执行被挂起与被恢复。是非抢占式的多任务子例程的概括，可以允许有多个入口点在例程中确定的位置来控制程序的暂停与恢复执行。通过编程方法态实现，协程切换任务由应用层面实现，开销很小。同个调度器下的协程应该是在同一个线程内。协同运行的例程，它是比是线程（thread）更细量级的用户态线程，特点是允许用户的主动调用和主动退出，挂起当前的例程然后返回值或去执行其他任务，接着返回原来停下的点继续执行。

**协程切换的代价比线程切换低。**

**协程切换**：只涉及基本的CPU上下文切换。把当前协程的 CPU 寄存器状态保存起来，然后将需要切换进来的协程的 CPU 寄存器状态加载的 CPU 寄存器上就 ok 了，而且完全在用户态进行。

**线程切换**：系统内核调度的对象是线程，因为线程是调度的基本单元（进程是资源拥有的基本单元，进程的切换需要做的事情更多），而线程的调度只有拥有最高权限的内核空间才可以完成，所以线程的切换涉及到用户空间和内核空间的切换，也就是特权模式切换，然后需要操作系统调度模块完成线程调度（taskstruct），而且除了和协程相同基本的 CPU 上下文，还有线程私有的栈和寄存器等，说白了就是上下文比协程多一些，而且特权模式切换的开销确实不小。

线程切换需要借助内核完成，一次用户态到内核态的切换，以及一次内核态到用户态的切换。而协程的切换只在用户态就可以完成，无需借助内核，也就不需要进入内核态。用户态和内核态的切换才是最主要的开销。

### 示例

```python
import time


def display(num):
    time.sleep(1)
    print(num)


for num in range(10):
    display(num)
```

上面这段代码，程序会每隔1秒中输出0到9的数字，因此整个程序的执行需要大约10秒时间。值得注意的是，因为没有使用多线程或多进程，程序中只有一个执行单元，而`time.sleep(1)`的休眠操作会让整个线程停滞1秒钟，对于上面的代码来说，在这段时间里面CPU是完全闲置的没有做什么事情。

在协程中，可以使用`await`让一个阻塞的子程序将CPU让给与它协作的子程序，定义一个异步函数。

```python
import asyncio


async def display(num):
    await asyncio.sleep(1)
    print(num)
```

异步函数不同于普通函数，调用普通函数会得到返回值，而调用异步函数会得到一个协程对象。我们需要将协程对象放到一个事件循环中才能达到与其他协程对象协作的效果，因为事件循环会负责处理子程序切换的操作，简单的说就是让阻塞的子程序让出CPU给可以执行的子程序。

通过下面的列表生成式来代码10个协程对象，跟刚才在循环中调用display函数的道理一致。

```python
coroutines = [display(num) for num in range(10)]
```

通过下面的代码可以获取事件循环并将协程对象放入事件循环中。

```python
loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait(coroutines))
loop.close()
```

执行上面的代码会发现，10个分别会阻塞1秒钟的协程总共只阻塞了约1秒种的时间，这就说明**协程对象一旦阻塞会将CPU让出而不是让CPU处于闲置状态**，这样就大大的**提升了CPU的利用率**。而且我们还会注意到，0到9的数字并不是按照我们创建协程对象的顺序打印出来的，这正是我们想要的结果啊；另外，多次执行该程序会发现每次输出的结果都不太一样，这正是并发程序本身执行顺序不确定性造成的结果。

***

## 基于生成器的协程

基于生成器的协程（generator-based coroutine）特指使用 `yield from` 句法的协程。

1. 生成器用于生成供迭代的数据
2. 协程是数据的消费者

### yield/send

Python对协程的支持是通过generator实现的。

在generator中，我们不但可以通过`for`循环来迭代，还可以不断调用`next()`函数获取由`yield`语句返回的下一个值。但是Python的`yield`不但可以返回一个值，它还可以接收调用者发出的参数。当协程执行到yield关键字时，会暂停在那一行，等到主线程调用send方法发送了数据，协程才会接到数据继续执行。

每个生成器都可以执行send()方法，为生成器内部的yield语句发送数据。此时yield语句不再只是`yield xxxx`的形式，还可以是`var = yield xxxx`的赋值形式。它同时具备两个功能，一是暂停并返回函数，二是接收外部send()方法发送过来的值，重新激活函数，并将这个值赋值给var变量！从逻辑上讲，用于实现协程的 yield 除了用于和调用者互相传递数据外，主要用于控制代码流程 。

协程可以处于下面四个状态中的一个。当前状态可以导入inspect模块，使用inspect.getgeneratorstate(...) 方法查看，该方法会返回下述字符串中的一个。

- 'GEN_CREATED'　　等待开始执行。
- 'GEN_RUNNING'　　协程正在执行。
- 'GEN_SUSPENDED'  在yield表达式处暂停。
- 'GEN_CLOSED' 　　执行结束。

刚刚创建的协程处于 `GEN_CREATED` 状态中，调用者需要使用 `next(my_coro)` 或者`my_coro.send(None)` 启动/激活这个协程，被激活的协程开始运行，直到第一个 `yield` 表达式，之后这个协程就准备好作用活跃的协程使用了（这一步通常称为 “预激” 协程）。

```python
>>> def coroutine():
...     reply = yield 'hello'
...     yield reply
... 

>>> c = coroutine()

>>> next(c)
'hello'

>>> c.send('world')
'world'
```

```python
def printer(): 
    counter = 0
    while True:
        string = str(yield)
        print('[{0}] {1}'.format(counter, string))
        counter += 1
 
if __name__ == '__main__':
    p = printer()
    next(p)
    p.send('Hi')
    p.send('My name is hsfzxjy.')
    p.send('Bye!')
```

仅当协程处于暂停状态时才能调用 send()方法，例如`my_coro.send(10)`。不过，如果协程还没激活（状态是`'GEN_CREATED'`），就立即把None之外的值发给它，会出现TypeError。因此，始终要先调用`next(my_coro)`激活协程（也可以调用`my_coro.send(None)`），这一过程被称作预激活。

```python
def consumer():
    r = ''
    while True:
        n = yield r		# 消费者通过yield接收生产者的消息，同时返给其结果
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)
        r = '200 OK'	# 消费者消费结果，下个循环返回给生产者

def produce(c):
    c.send(None)		# 启动消费者生成器，同时第一次接收返回结果
    n = 0
    while n < 5:
        n = n + 1
        print('[PRODUCER] Producing %s...' % n)
        r = c.send(n)	# 向消费者发送消息并准备接收结果。此时会切换到消费者执行
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()			# 关闭消费者生成器

c = consumer()
produce(c)
```

注意到`consumer`函数是一个`generator`，把一个`consumer`传入`produce`后：

1. 首先调用`c.send(None)`启动生成器；
2. 然后，一旦生产了东西，通过`c.send(n)`切换到`consumer`执行；
3. `consumer`通过`yield`拿到消息，处理，又通过`yield`把结果传回；
4. `produce`拿到`consumer`处理的结果，继续生产下一条消息；
5. `produce`决定不生产了，通过`c.close()`关闭`consumer`，整个过程结束。

整个流程无锁，由一个线程执行，`produce`和`consumer`协作完成任务，所以称为“协程”，而非线程的抢占式多任务。

协程的特点在于是一个线程执行，那和多线程比，协程有何优势？

最大的优势就是协程极高的执行效率。协程不是被操作系统内核所管理，而完全是由程序所控制（也就是在用户态执行）。因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销，和多线程比，线程数量越多，协程的性能优势就越明显。

第二大优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不加锁，只需要判断状态就好了，所以执行效率比多线程高很多。

因为协程是一个线程执行，那怎么利用多核CPU呢？最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

### yield from

yield from 其实就是等待另外一个协程的返回。主要解决的就是在生成器里嵌套生成器不方便的问题。它有两大主要功能:

* 让嵌套生成器不必通过循环迭代`yield`，而是直接`yield from`。`yield from iterable`本质上是`for item in iterable: yield item`的缩写版  。
* 在子生成器和委派生成器的调用者之间打开双向通道，两者可以直接通信。

生成器可以使用 `yield from` 句法将部分操作委托给另一个生成器，也就是说，可以将包含 `yield` 的代码逻辑重构为另一个生成器。 `yield from` 句法会创建一个「通道」，把内层生成器（子生成器）与外层生成器的客户端联系起来，这样二者可以直接发送和产出值，还可以直接传入异常。这样一来，就省去了之前把生成器的工作委托给子生成器所需要的大量样板代码。

```python
def gen():
    yield from subgen()

def subgen():
    while True:
        x = yield
        yield x+1

def main():
    g = gen()
    next(g)                # 驱动生成器g开始执行到第一个 yield
    retval = g.send(1)     # 看似向生成器 gen() 发送数据
    print(retval)          # 返回2
```

专门的术语：

- 委派生成器 （delegating generator）- 包含 `yield from <iterable>` 表达式的生成器函数。
- 子生成器 （subgenerator）- 从 `<iterable>` 部分获取的生成器。
- 调用方 （caller）- 调用委派生成器的客户端代码。

同时，PEP 380 中对 `yield from` 的行为做了如下说明：

- 子生成器产出的值都直接传给委派生成器的调用方。
- 使用 `send()` 方法发给委派生成器的值都直接传给子生成器。如果发送的值是 `None` ，那么会调用子 生成器的 `__next__()` 方法。如果发送的值不是 `None` ，那么会调用子生成器的 `send()` 方法。如 果子生成器抛出 `StopIteration` 异常，那委派生成器恢复运行。任何其它异常都会向上冒泡，传给委派生 成器。
- 生成器退出时，生成器（或子生成器）中的 `return expr` 表达式会触发 `StopIteration(expr)` 异常 抛出。
- `yield from` 表达式的值是子生成器终止时传给 `StopIteration` 异常的第一个参数值。
- 传入委派生成器的异常，除了 `GeneratorExit` 之外都传给子生成器的 `throw()` 方法。如果调用 `throw()` 方法时抛出了 `StopIteration` 异常，委派生成器恢复运行。 `StopIteration` 之外的异 常会向上冒泡，传给委派生成器。
- 如果把 `GeneratorExit` 异常传入委派生成器，或者在委派生成器上调用 `close()` 方法，那么（委派 生成器）调用子生成器的 `close()` 方法（如果子生成器提供了该方法的话）。如果子生成器的 `close()` 方法导致异常抛出，那么异常会向上冒泡，传给委派生成器。如果子生成器的 `close()` 方法 正常执行的话，委派生成器随后向上抛出 `GeneratorExit` 异常。

### Coroutine与Generator

由于两者在语言层面并没有明显的区别，将生成器函数视为迭代器还是协程，完全依赖使用场景和上下文。

最重要的区别：

- generator生成供迭代的数据
- coroutine数据(data)的消费者
- coroutine不会与迭代操作关联，而generator会
- coroutine强调协同控制程序流，generator强调保存状态和产生数据

相似的是，它们都是不用return来实现重复调用的函数/对象，都用到了yield(中断/恢复)的方式来实现。

***

## 原生协程

用yield from容易在表示协程和生成器中混淆，没有良好的语义性，所以在Python 3.5推出了更新的async/await表达式来作为协程的语法，以便从句法上和基于生成器的协程进行区分。很多人仍然弄不明白生成器和协程的联系与区别，也弄不明白`yield` 和 `yield from` 的区别。这种混乱的状态也违背Python之禅的一些准则。

同时，为了让两种生成器可以互相操作，Python 3.5 还提供了 `types.coroutine`装饰器函数对基于生成器的协程进行了包装。打算交给 *asyncio* 处理基于生成器的协程 **最好** 使用 `asyncio.coroutine` （Python 3.4）或者`types.coroutine` （Python 3.5）进行装饰，这样可以把协程函数凸显出来，也有助于调试

原生协程（native coroutine）有如下关键特征：

- `async def` functions are always coroutines, even if they do not contain `await` expressions.

- It is a `SyntaxError` to have `yield` or `yield from` expressions in an `async def` function.

- Internally, two new code object flags were introduced:

  > - `CO_COROUTINE` is used to mark *native coroutines* (defined with new syntax).
  > - `CO_ITERABLE_COROUTINE` is used to make *generator-based coroutines* compatible with *native coroutines*.

- Regular generators, when called, return a *generator object*; similarly, coroutines return a *coroutine object*.

- `StopIteration` exceptions are not propagated out of coroutines, and are replaced with a `RuntimeError`. For regular generators such behavior requires a future import (see PEP 479).

- When a *native coroutine* is garbage collected, a `RuntimeWarning` is raised if it was never awaited on (see also Debugging Features).

原生协程（native coroutine）和 基于生成器的协程（generator-based coroutine）有如下区别：

- 原生协程对象并未实现 `__iter__` 和 `__next__` 方法，所以它们并不能用于需要可迭代对象的场景。
- 普通生成器函数内部不能使用 `yield from 原生协程` 句法。
- 基于生成器的协程函数（代码用于 *asyncio* 时，需用 `asyncio.coroutine` ）内部可以使用 `yield from 原生协程` 句法。
- `inspect.isgenerator()` 函数和 `inspect.isgeneratorfunction()` 函数作用于原生协程时，返回 `False` 。

```python
from time import sleep

# 生成器 - 数据生产者
def countdown_gen(n, consumer):
    consumer.send(None)
    while n > 0:
        consumer.send(n)
        n -= 1
    consumer.send(None)


# 协程 - 数据消费者
def countdown_con():
    while True:
        n = yield
        if n:
            print(f'Countdown {n}')
            sleep(1)
        else:
            print('Countdown Over!')


def main():
    countdown_gen(5, countdown_con())


if __name__ == '__main__':
    main()
```

上面代码中`countdown_gen`函数中的第1行`consumer.send(None)`是为了激活生成器，通俗的说就是让生成器执行到有`yield`关键字的地方挂起，当然也可以通过`next(consumer)`来达到同样的效果。如果不愿意每次都用这样的代码来“预激”生成器，可以写一个包装器来完成该操作，代码如下所示。

```python
from functools import wraps


def coroutine(fn):

    @wraps(fn)
    def wrapper(*args, **kwargs):
        gen = fn(*args, **kwargs)
        next(gen)
        return gen

    return wrapper
```

下面观察一个asyncio中Future的例子：

```python
import asyncio

future = asyncio.Future()

async def coro1():
    await asyncio.sleep(1)
    future.set_result('data')

async def coro2():
    print(await future)

loop = asyncio.get_event_loop()
loop.run_until_complete(asyncio.wait([
    coro1(), 
    coro2()
]))
loop.close()
```

两个协程在在事件循环中，协程coro1在执行第一句后挂起自身切到asyncio.sleep，而协程coro2一直等待future的结果，让出事件循环，计时器结束后coro1执行了第二句设置了future的值，被挂起的coro2恢复执行，打印出future的结果'data'。

用`asyncio`的异步网络连接来获取sina、sohu和163的网站首页：

```python
import asyncio

@asyncio.coroutine
def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = yield from connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    yield from writer.drain()
    while True:
        line = yield from reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

改写：

```python
import asyncio

async def wget(host):
    print('wget %s...' % host)
    connect = asyncio.open_connection(host, 80)
    reader, writer = await connect
    header = 'GET / HTTP/1.0\r\nHost: %s\r\n\r\n' % host
    writer.write(header.encode('utf-8'))
    await writer.drain()
    while True:
        line = await reader.readline()
        if line == b'\r\n':
            break
        print('%s header > %s' % (host, line.decode('utf-8').rstrip()))
    # Ignore the body, close the socket
    writer.close()

loop = asyncio.get_event_loop()
tasks = [wget(host) for host in ['www.sina.com.cn', 'www.sohu.com', 'www.163.com']]
loop.run_until_complete(asyncio.wait(tasks))
loop.close()
```

***

## 协程

在Python语言中，单线程+异步I/O的编程模型称为协程。协程最大的优势就是极高的执行效率，因为子程序切换不是线程切换，而是由程序自身控制，因此，没有线程切换的开销。协程的第二个优势就是不需要多线程的锁机制，因为只有一个线程，也不存在同时写变量冲突，在协程中控制共享资源不用加锁，只需要判断状态就好了，所以执行效率比多线程高很多。如果想要充分利用CPU的多核特性，最简单的方法是多进程+协程，既充分利用多核，又充分发挥协程的高效率，可获得极高的性能。

几乎所有的异步框架都将异步编程模型简化：一次只允许处理一个事件。故而有关异步的讨论几乎都集中在了单线程内。如果某事件处理程序需要长时间执行，所有其他部分都会被阻塞。所以，一旦采取异步编程，每个异步调用必须"足够小"，不能耗时太久。如何拆分异步任务成了难题。

- 程序下一步行为往往依赖上一步执行结果，如何知晓上次异步调用已完成并获取结果？
- 回调（Callback）成了必然选择。那又需要面临“回调地狱”的折磨。
- 同步代码改为异步代码，必然破坏代码结构。
- 解决问题的逻辑也要转变，不再是一条路走到黑，需要精心安排异步任务。

在事件循环+回调的基础上衍生出了基于协程的解决方案，代表作有 Tornado、Twisted、asyncio 等。异步编程最大的困难：异步任务何时执行完毕？接下来要对异步调用的返回结果做什么操作？程序得知道当前所处的状态，而且要将这个状态在不同的回调之间延续下去。任务之间得相互通知，每个任务得有自己的状态。那不就是很古老的编程技法：协作式多任务？然而要在单线程内做调度，啊哈，协程！每个协程具有自己的栈帧，当然能知道自己处于什么状态，协程之间可以协作那自然可以通知别的协程。

不用回调的方式了，怎么知道异步调用的结果呢？先设计一个对象，异步调用执行完的时候，就把结果放在它里面。这种对象称之为Future对象。

Future对象有一个`result`属性，用于存放未来的执行结果。还有个`set_result()`方法，是用于设置`result`的，并且会在给`result`绑定值以后运行事先给`future`添加的回调。回调是通过Future对象的`add_done_callback()`方法添加的。

不要疑惑此处的`callback`，说好了不回调的嘛？难道忘了我们曾经说的要异步，必回调。不过也别急，此处的回调，和先前学到的回调，还真有点不一样。`add_done_callback()`不是写业务逻辑用的，此前的`callback`可就干的是业务逻辑呀。生成器，其内部写完了所有的业务逻辑。

遵循一个编程规则：单一职责，每种角色各司其职，如果还有工作没有角色来做，那就创建一个角色去做。Task封装了`coro`对象，即初始化时传递给他的对象，被管理的任务是待执行的协程。我们知道生成器需要先调用`next()`迭代一次或者是先`send(None)`启动，遇到`yield`之后便暂停。Task 在初始化的时候就会执行一遍，`step()`内会调用生成器的`send()`方法，初始化第一次发送的是`None`就驱动了`coro`的第一次执行。这样Task, Future, Coroutine三者精妙地串联在了一起。

初始化`Task`对象以后，把Coroutine给驱动到了`yied `就完事了，接下来怎么继续？

该事件循环上场了。事件循环(Event Loop)驱动协程运行。事件循环就像心脏一般，只要它开始跳动，整个程序就会持续运行。

因为协程能够保存自己的状态，知道自己的future是哪个。也不用关心到底要设置什么值，因为要设置什么值也是协程内安排的。这里的回调根本不关心是谁触发了这个事件，以及`future`是谁它也不关心。

**生成器协程风格VS回调风格对比总结：**

在回调风格中：

- 存在链式回调（虽然示例中嵌套回调只有一层）
- 请求和响应也不得不分为两个回调以至于破坏了同步代码那种结构
- 程序员必须在回调之间维护必须的状态。

而基于生成器协程的风格：

- 无链式调用
- 回调里只管给`future`设置值，不再关心业务逻辑
- `loop` 内回调`callback()`不再关注是谁触发了事件
- 已趋近于同步代码的结构
- 无需程序员在多个协程之间维护状态

`asyncio`是Python 3.4 试验性引入的异步I/O框架（PEP 3156），提供了基于协程做异步I/O编写单线程并发代码的基础设施。其核心组件有事件循环（Event Loop）、协程(Coroutine）、任务(Task)、未来对象(Future)以及其他一些扩充和辅助性质的模块。

对比生成器版的协程，使用asyncio库后变化很大：

- 没有了`yield` 或 `yield from`，而是`async/await`
- 没有了自造的`loop()`，取而代之的是`asyncio.get_event_loop()`
- 没有了显式的 `Future` 和 `Task`，`asyncio`已封装
- 更少量的代码，更优雅的设计

协程的本质是个单线程,它不能同时将 单个CPU 的多个核用上,协程需要和进程配合才能运行在多CPU上.当然我们日常所编写的绝大部分应用都没有这个必要，除非是cpu密集型应用。进行阻塞（Blocking）操作（如IO时）会阻塞掉整个程序。

PS：

**协程**是计算机程序的一类组件，推广了非抢先多任务的子程序，允许执行被挂起与被恢复。是非抢占式的多任务子例程的概括，可以允许有多个入口点在例程中确定的位置来控制程序的暂停与恢复执行。相对子例程而言，协程更为一般和灵活。协程更适合于用来实现彼此熟悉的程序组件，如合作式多任务、异常处理、事件循环、迭代器、无限列表和管道。 

**子程序**是一个大型程序中的某部分代码，由一个或多个语句块组成。它负责完成某项特定任务，而且相较于其他代码，具备相对的独立性。子程序（subroutine）是一个概括性的术语，子程序（subroutine）是所有高端程序所称。它经常被使用在汇编语言层级上。子程序的主体（body）是一个代码区块，当它被调用时就会进入运行。

**协程VS子程序**：

子例程的起始处是惟一的入口点，一旦退出即完成了子例程的执行，子例程的一个实例只会返回一次。

协程可以通过yield来调用其它协程。通过yield方式转移执行权的协程之间不是调用者与被调用者的关系，而是彼此对称、平等的。协程的起始处是第一个入口点，在协程里，返回点之后是接下来的入口点。子例程的生命期遵循后进先出（最后一个被调用的子例程最先返回）；相反，协程的生命期完全由他们的使用的需要决定。 

因为相对于子例程，协程可以有多个入口和出口点，可以用协程来实现任何的子例程。事实上，正如Knuth所说：“子例程是协程的特例。”

每当子例程被调用时，执行从被调用子例程的起始处开始；然而，接下来的每次协程被调用时，从协程返回（或yield）的位置接着执行。

因为子例程只返回一次，要返回多个值就要通过集合的形式。这在有些语言，如Forth里很方便；而其他语言，如C，只允许单一的返回值，所以就需要引用一个集合。相反地，因为协程可以返回多次，返回多个值只需要在后继的协程调用中返回附加的值即可。在后继调用中返回附加值的协程常被称为产生器。

子例程容易实现于堆栈之上，因为子例程将调用的其他子例程作为下级。相反地，协程对等地调用其他协程，最好的实现是用continuations（由有垃圾回收的堆实现）以跟踪控制流程。 

***

## 事件驱动

```python
import asyncio

async def compute(x, y):
    print("Compute %s + %s ..." % (x, y))
    await asyncio.sleep(1.0)
    return x + y

async def print_sum(x, y):
    result = await compute(x, y)
    print("%s + %s = %s" % (x, y, result))

loop = asyncio.get_event_loop()
loop.run_until_complete(print_sum(1, 2))
loop.close()
```

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20Event%20loop-1.png)

当事件循环开始运行时，它会在Task中寻找coroutine来执行调度，因为事件循环注册了*print_sum()*，因此*print_sum()*被调用，执行*result = await compute(x, y)*这条语句（等同于*result = yield from compute(x, y)*），因为*compute()*自身就是一个coroutine，因此*print_sum()*这个协程就会暂时被挂起，*compute()*被加入到事件循环中，程序流执行*compute()*中的print语句，打印”Compute %s + %s …”，然后执行了*await asyncio.sleep(1.0)*，因为*asyncio.sleep()*也是一个coroutine，接着*compute()*就会被挂起，等待计时器读秒，在这1秒的过程中，事件循环会在队列中查询可以被调度的coroutine，而因为此前*print_sum()*与*compute()*都被挂起了，因此事件循环会停下来等待协程的调度，当计时器读秒结束后，程序流便会返回到*compute()*中执行return语句，结果会返回到*print_sum()*中的result中，最后打印result，事件队列中没有可以调度的任务了，此时*loop.close()*把事件队列关闭，程序结束。

让我们用例子来比较和对比一下单线程、多线程以及事件驱动编程模型。下图展示了随着时间的推移，这三种模式下程序所做的工作。这个程序有3个任务需要完成，每个任务都在等待I/O操作时阻塞自身。阻塞在I/O操作上所花费的时间已经用灰色框标示出来了。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%203%20comparison.png)

在单线程同步模型中，任务按照顺序执行。如果某个任务因为I/O而阻塞，其他所有的任务都必须等待，直到它完成之后它们才能依次执行。这种明确的执行顺序和串行化处理的行为是很容易推断得出的。如果任务之间并没有互相依赖的关系，但仍然需要互相等待的话这就使得程序不必要的降低了运行速度。

![](https://note-taking-1258869021.cos.ap-beijing.myqcloud.com/python/thread%20process%20Event%20loop-2.png)

在多线程版本中，这3个任务分别在独立的线程中执行。这些线程由操作系统来管理，在多处理器系统上可以并行处理，或者在单处理器系统上交错执行。这使得当某个线程阻塞在某个资源的同时其他线程得以继续执行。与完成类似功能的同步程序相比，这种方式更有效率，但程序员必须写代码来保护共享资源，防止其被多个线程同时访问。多线程程序更加难以推断，因为这类程序不得不通过线程同步机制如锁、可重入函数、线程局部存储或者其他机制来处理线程安全问题，如果实现不当就会导致出现微妙且令人痛不欲生的bug。

在事件驱动版本的程序中，3个任务交错执行，但仍然在一个单独的线程控制中。当处理I/O或者其他昂贵的操作时，注册一个回调到事件循环中，然后当I/O操作完成时继续执行。回调描述了该如何处理某个事件。事件循环轮询所有的事件，当事件到来时将它们分配给等待处理事件的回调函数。这种方式让程序尽可能的得以执行而不需要用到额外的线程。事件驱动型程序比多线程程序更容易推断出行为，因为程序员不需要关心线程安全问题。

当我们面对如下的环境时，事件驱动模型通常是一个好的选择：

1. 程序中有许多任务
2. 任务之间高度独立（因此它们不需要互相通信，或者等待彼此）
3. 在等待事件到来时，某些任务会阻塞。

当应用程序需要在任务间共享可变的数据时，这也是一个不错的选择，因为这里不需要采用同步处理。

网络应用程序通常都有上述这些特点，这使得它们能够很好的契合事件驱动编程模型。

***

参考：

[协程](https://www.liaoxuefeng.com/wiki/0014316089557264a6b348958f449949df42a6d3a2e542c000/001432090171191d05dae6e129940518d1d6cf6eeaaa969000)，廖雪峰

[从0到1，Python异步编程的演进之路](https://zhuanlan.zhihu.com/p/25228075)，tonnie

[编程语言 - Python 3 的 AsyncIO 库](https://ialloc.org/blog/into-python3-asyncio/)

[深入理解Python异步编程](http://aju.space/2017/07/31/Drive-into-python-asyncio-programming-part-1.html)，aju

[多进程、协程、事件驱动及select poll epoll](http://www.cnblogs.com/zhaof/p/5932461.html)

[[译]基于异步协程的网络爬虫](http://kissg.me/2016/06/01/a-web-crawler-with-asyncio-coroutines/)，kissg

[并发下载](https://github.com/jackfrued/Python-100-Days/blob/master/Day66-75/69.%E5%B9%B6%E5%8F%91%E4%B8%8B%E8%BD%BD.md)，骆昊

[为什么协程切换的代价比线程切换低?](https://www.zhihu.com/question/308641794/answer/572499202)，暗淡了乌云、张凯(Kyle Zhang)

[出于什么样的原因，诞生了「协程」这一概念？](https://www.zhihu.com/question/50185085/answer/183463734)，PaintDream