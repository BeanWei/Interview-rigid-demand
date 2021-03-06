# [Python之线程、进程和协程](http://python.jobbole.com/86406/)

## 引言
解释器环境：python3.5.1
我们都知道python网络编程的两大必学模块socket和socketserver，其中的socketserver是一个支持IO多路复用和多线程、多进程的模块。一般我们在socketserver服务端代码中都会写这么一句：
`server = socketserver.ThreadingTCPServer(settings.IP_PORT, MyServer)`
ThreadingTCPServer这个类是一个支持多线程和TCP协议的socketserver，它的继承关系是这样的：
`class ThreadingTCPServer(ThreadingMixIn, TCPServer): pass`
右边的TCPServer实际上是它主要的功能父类，而左边的ThreadingMixIn则是实现了多线程的类，它自己本身则没有任何代码。
MixIn在python的类命名中，很常见，一般被称为“混入”，戏称“乱入”，通常为了某种重要功能被子类继承。
```
class ThreadingMixIn:
     
    daemon_threads = False
 
    def process_request_thread(self, request, client_address):      
        try:
            self.finish_request(request, client_address)
            self.shutdown_request(request)
        except:
            self.handle_error(request, client_address)
            self.shutdown_request(request)
 
    def process_request(self, request, client_address):
         
        t = threading.Thread(target = self.process_request_thread,
                             args = (request, client_address))
        t.daemon = self.daemon_threads
        t.start()
```
在ThreadingMixIn类中，其实就定义了一个属性，两个方法。在process_request方法中实际调用的正是python内置的多线程模块threading。这个模块是python中所有多线程的基础，socketserver本质上也是利用了这个模块。

## 一、线程
线程，有时被称为轻量级进程(Lightweight Process，LWP），是程序执行流的最小单元。一个标准的线程由线程ID，当前指令指针(PC），寄存器集合和堆栈组成。另外，线程是进程中的一个实体，是被系统独立调度和分派的基本单位，线程自己不独立拥有系统资源，但它可与同属一个进程的其它线程共享该进程所拥有的全部资源。一个线程可以创建和撤消另一个线程，同一进程中的多个线程之间可以并发执行。由于线程之间的相互制约，致使线程在运行中呈现出间断性。线程也有就绪、阻塞和运行三种基本状态。就绪状态是指线程具备运行的所有条件，逻辑上可以运行，在等待处理机；运行状态是指线程占有处理机正在运行；阻塞状态是指线程在等待一个事件（如某个信号量），逻辑上不可执行。每一个应用程序都至少有一个进程和一个线程。线程是程序中一个单一的顺序控制流程。在单个程序中同时运行多个线程完成不同的被划分成一块一块的工作，称为多线程。
以上那一段，可以不用看！举个例子，厂家要生产某个产品，在它的生产基地建设了很多厂房，每个厂房内又有多条流水生产线。所有厂房配合将整个产品生产出来，某个厂房内的所有流水线将这个厂房负责的产品部分生产出来。每个厂房拥有自己的材料库，厂房内的生产线共享这些材料。而每一个厂家要实现生产必须拥有至少一个厂房一条生产线。那么这个厂家就是某个应用程序；每个厂房就是一个进程；每条生产线都是一个线程。

### 1.1 普通的多线程
在python中，threading模块提供线程的功能。通过它，我们可以轻易的在进程中创建多个线程。下面是个例子：
```
import threading
import time
   
def show(arg):
    time.sleep(1)
    print('thread'+str(arg))
   
for i in range(10):
    t = threading.Thread(target=show, args=(i,))
    t.start()
   
print('main thread stop')
```

上述代码创建了10个“前台”线程，然后控制器就交给了CPU，CPU根据指定算法进行调度，分片执行指令。
下面是Thread类的主要方法：
* start 线程准备就绪，等待CPU调度
* setName 为线程设置名称
* getName 获取线程名称
* setDaemon 设置为后台线程或前台线程（默认是False，前台线程）
    如果是后台线程，主线程执行过程中，后台线程也在进行，主线程执行完毕后，后台线程不论成功与否，均停止。如果是前台线程，主线程执行过程中，前台线程也在进行，主线程执行完毕后，等待前台线程也执行完成后，程序停止。
* join 该方法非常重要。它的存在是告诉主线程，必须在这个位置等待子线程执行完毕后，才继续进行主线程的后面的代码。但是当setDaemon为True时，join方法是无效的。
* run 线程被cpu调度后自动执行线程对象的run方法

### 1.2 自定义线程类
对于threading模块中的Thread类，本质上是执行了它的run方法。因此可以自定义线程类，让它继承Thread类，然后重写run方法。
```
import threading
class MyThreading(threading.Thread):
 
    def __init__(self,func,arg):
        super(MyThreading,self).__init__()
        self.func = func
        self.arg = arg
 
    def run(self):
        self.func(self.arg)
 
def f1(args):
    print(args)
 
obj = MyThreading(f1, 123)
obj.start()
```

### 1.3 线程锁
CPU执行任务时，在线程之间是进行随机调度的，并且每个线程可能只执行n条代码后就转而执行另外一条线程。由于在一个进程中的多个线程之间是共享资源和数据的，这就容易造成资源抢夺或脏数据，于是就有了锁的概念，限制某一时刻只有一个线程能访问某个指定的数据。

### 未使用锁
```
#coding:utf-8
import threading
import time

NUM = 0

def show():
    global NUM
    NUM += 1
    name = t.getName()
    print(name, '执行完毕后，NUM的值为：', NUM)

for i in range(10):
    t = threading.Thread(target=show)
    t.start()

```
输出结果:
```
Thread-1 执行完毕后，NUM的值为：  10
Thread-2 执行完毕后，NUM的值为：  10
Thread-4 执行完毕后，NUM的值为：  10
Thread-9 执行完毕后，NUM的值为：  10
Thread-3 执行完毕后，NUM的值为：  10
Thread-6 执行完毕后，NUM的值为：  10
Thread-8 执行完毕后，NUM的值为：  10
Thread-7 执行完毕后，NUM的值为：  10
Thread-5 执行完毕后，NUM的值为：  10
Thread-10 执行完毕后，NUM的值为：  10
```

由此可见，由于线程同时访问一个数据，产生了错误的结果。为了解决这个问题，python在threading模块中定义了几种线程锁类，分别是：
* Lock 普通锁（不可嵌套）
* RLock 普通锁（可嵌套）常用
* Semaphore 信号量
* event 事件
* condition 条件

### 1.3.2 普通锁Lock和RLock
类名：Lock或RLock
普通锁，也叫互斥锁，是独占的，同一时刻只有一个线程被放行。
```
import time
import threading
 
NUM = 10
def func(lock):
    global NUM
    lock.acquire()  # 让锁开始起作用
    NUM -= 1
    time.sleep(1)
    print(NUM)
    lock.release() # 释放锁
 
lock = threading.Lock()   # 实例化一个锁对象
for i in range(10):
    t = threading.Thread(target=func, args=(lock,))  # 记得把锁当作参数传递给func参数
    t.start()
```
以上是threading模块的Lock类，它不支持嵌套锁。RLcok类的用法和Lock一模一样，但它支持嵌套，因此我们一般直接使用RLcok类。

### 1.3.3 信号量(Semaphore)
类名：BoundedSemaphore
这种锁允许一定数量的线程同时更改数据，它不是互斥锁。比如地铁安检，排队人很多，工作人员只允许一定数量的人进入安检区，其它的人继续排队。
```
import time
import threading
 
def run(n):
    semaphore.acquire()
    print('run the Thread: %s' % n)
    time.sleep(1)
    semaphore.release()

num = 0
semaphore = threading.BoundedSemaphore(5)
for i in range(20):
    t = threading.Thread(target=run, args=(i,))
    t.start()
```
输出结果:
```
run the Thread: 0
run the Thread: 1
run the Thread: 2
run the Thread: 3
run the Thread: 4
run the Thread: 5
run the Thread: 6
run the Thread: 7
run the Thread: 8
run the Thread: 9
run the Thread: 10
run the Thread: 12
run the Thread: 13
run the Thread: 11
run the Thread: 14
run the Thread: 15
run the Thread: 16
run the Thread: 18
run the Thread: 19
run the Thread: 17
```

### 1.3.4 事件(Event)
类名：Event
事件主要提供了三个方法 set、wait、clear。
事件机制：全局定义了一个“Flag”，如果“Flag”的值为False，那么当程序执行wait方法时就会阻塞，如果“Flag”值为True，那么wait方法时便不再阻塞。这种锁，类似交通红绿灯（默认是红灯），它属于在红灯的时候一次性阻挡所有线程，在绿灯的时候，一次性放行所有的排队中的线程。
clear：将“Flag”设置为False
set：将“Flag”设置为True

```
import threading

def func(e, i):
    print(i)
    e.wait()
    print(i+100)

event = threading.Event()

for i in range(10):
    t = threading.Thread(target=func, args=(event, i))
    t.start()

event.clear()
inp = input('>>>')
if inp == '1':
    event.set()
```
输出结果：
```
0
1
2
3
4
5
6
7
8
9
>>>1
101
103
100
106
108
102
105
109
107
104
```

### 1.3.5 条件(condition)
类名：Condition
该机制会使得线程等待，只有满足某条件时，才释放n个线程。

### 1.3 全局解释器锁（GIL）
[Python 最难的问题](https://www.oschina.net/translate/pythons-hardest-problem)

### 1.4 定时器（Timer）
定时器，指定n秒后执行某操作。很简单但很实用的东西。
```
from threading import Timer
def hello():
    print("hello, world")
t = Timer(1, hello)  # 表示1秒后执行hello函数
t.start() 
```

### 1.5 队列
通常而言，队列是一种先进先出的数据结构，与之对应的是堆栈这种后进先出的结构。但是在python中，它内置了一个queue模块，它不但提供普通的队列，还提供一些特殊的队列。具体如下：
* queue.Queue ：先进先出队列
* queue.LifoQueue ：后进先出队列
* queue.PriorityQueue ：优先级队列
* queue.deque ：双向队列

### 1.5.1 Queue：先进先出队列
这是最常用也是最普遍的队列，先看一个例子。
```
import queue
q = queue.Queue(5)
q.put(11)
q.put(22)
q.put(33)
 
print(q.get())
print(q.get())
print(q.get())
```

***Queue类的参数和方法：***
* maxsize 队列的最大元素个数，也就是queue.Queue(5)中的5。当队列内的元素达到这个值时，后来的元素默认会阻塞，等待队列腾出位置。 
```
def __init__(self, maxsize=0):
self.maxsize = maxsize
self._init(maxsize)
```
* qsize() 获取当前队列中元素的个数，也就是队列的大小
* empty() 判断当前队列是否为空，返回True或者False
* full() 判断当前队列是否已满，返回True或者False
* put(self, block=True, timeout=None)
    >往队列里放一个元素，默认是阻塞和无时间限制的。如果，block设置为False，则不阻塞，这时，如果队列是满的，放不进去，就会弹出异常。如果timeout设置为n秒，则会等待这个秒数后才put，如果put不进去则弹出异常。

* get(self, block=True, timeout=None) 从队列里获取一个元素。参数和put是一样的意思。
* join() 阻塞进程，直到所有任务完成，需要配合另一个方法task_done。 
```
def join(self):
    with self.all_tasks_done:
        while self.unfinished_tasks:
            self.all_tasks_done.wait()
```
* task_done() 表示某个任务完成。每一条get语句后需要一条task_done。
```
import queue
q = queue.Queue(5)
q.put(11)
q.put(22)
print(q.get())
q.task_done()
print(q.get())
q.task_done()
q.join()
```

### 1.5.2 LifoQueue：后进先出队列
类似于“堆栈”，后进先出。也较常用。
```
>>> import queue
>>> q = queue.LifoQueue()
>>> q.put(11)
>>> q.put(22)
>>> q.get()
22
>>> q.get()
11
>>>
```

### 1.5.3 PriorityQueue：优先级队列
带有权重的队列，每个元素都是一个元组，前面的数字表示它的优先级，数字越小优先级越高，同样的优先级先进先出.
```
>>> import queue
>>> q = queue.PriorityQueue()
>>> q.put((1, "gao"))
>>> q.put((10, "di"))
>>> q.get()
(1, 'gao')
>>> q.get()
(10, 'di')
>>>
```

### 1.5.4 deque：双向队列
Queue和LifoQueue的“综合体”，双向进出。方法较多，使用复杂，慎用！
```
>>> import queue
>>> q = queue.deque()
>>> q.appendleft(0)
>>> q.append(5)
>>> q.append(8)
>>> q.pop()
8
>>> q.popleft()
0
```

### 1.6 生产者消费者模型
利用多线程和队列可以搭建一个生产者消费者模型，用于处理大并发的服务。
>    在并发编程中使用生产者和消费者模式能够解决绝大多数并发问题。该模式通过平衡生产线程和消费线程的工作能力来提高程序的整体处理数据的速度。

>    ***为什么要使用生产者和消费者模式***

>    在线程世界里，生产者就是生产数据的线程，消费者就是消费数据的线程。在多线程开发当中，如果生产者处理速度很快，而消费者处理速度很慢，那么生产者就必须等待消费者处理完，才能继续生产数据。同样的道理，如果消费者的处理能力大于生产者，那么消费者就必须等待生产者。为了解决这个问题于是引入了生产者和消费者模式。

>    ***什么是生产者消费者模式***

>    生产者消费者模式是通过一个容器来解决生产者和消费者的强耦合问题。生产者和消费者彼此之间不直接通讯，而通过阻塞队列来进行通讯，所以生产者生产完数据之后不用等待消费者处理，直接扔给阻塞队列，消费者不找生产者要数据，而是直接从阻塞队列里取，阻塞队列就相当于一个缓冲区，平衡了生产者和消费者的处理能力。

>   这个阻塞队列就是用来给生产者和消费者解耦的。纵观大多数设计模式，都会找一个第三者出来进行解耦，如工厂模式的第三者是工厂类，模板模式的第三者是模板类。在学习一些设计模式的过程中，如果先找到这个模式的第三者，能帮助我们快速熟悉一个设计模式。

>    以上摘自方腾飞的《聊聊并发——生产者消费者模式》

### 1.7 线程池

在使用多线程处理任务时也不是线程越多越好，由于在切换线程的时候，需要切换上下文环境，依然会造成cpu的大量开销。为解决这个问题，线程池的概念被提出来了。预先创建好一个较为优化的数量的线程，让过来的任务立刻能够使用，就形成了线程池。在python中，没有内置的较好的线程池模块，需要自己实现或使用第三方模块。


## 二、进程

在python中multiprocess模块提供了Process类，实现进程相关的功能。但是，由于它是基于fork机制的，因此不被windows平台支持。想要在windows中运行，必须使用if __name__ == '__main__:的方式，显然这只能用于调试和学习，不能用于实际环境。
（PS：在这里我必须吐槽一下python的包、模块和类的组织结构。在multiprocess中你既可以import大写的Process，也可以import小写的process，这两者是完全不同的东西。这种情况在python中很多，新手容易傻傻分不清。）
下面是一个简单的多进程例子，你会发现Process的用法和Thread的用法几乎一模一样。
```
from multiprocessing import Process
 
def foo(i):
    print("This is Process ", i)
 
if __name__ == '__main__':
    for i in range(5):
        p = Process(target=foo, args=(i,))
        p.start()
```

### 2.1 进程的数据共享
每个进程都有自己独立的数据空间，不同进程之间通常是不能共享数据，创建一个进程需要非常大的开销。
```
from multiprocessing import Process
list_1 = []
def foo(i):
    list_1.append(i)
    print("This is Process ", i," and list_1 is ", list_1)
 
if __name__ == '__main__':
    for i in range(5):
        p = Process(target=foo, args=(i,))
        p.start()
 
    print("The end of list_1:", list_1)
```
运行上面的代码，你会发现列表list_1在各个进程中只有自己的数据，完全无法共享。想要进程之间进行资源共享可以使用queues/Array/Manager这三个multiprocess模块提供的类。

### 2.1.1 使用Array共享数据
```
from multiprocessing import Process
from multiprocessing import Array
 
def Foo(i,temp):
    temp[0] += 100
    for item in temp:
        print(i,'----->',item)
 
if __name__ == '__main__':
    temp = Array('i', [11, 22, 33, 44])
    for i in range(2):
        p = Process(target=Foo, args=(i,temp))
        p.start()
```
对于Array数组类，括号内的“i”表示它内部的元素全部是int类型，而不是指字符i，列表内的元素可以预先指定，也可以指定列表长度。概括的来说就是Array类在实例化的时候就必须指定数组的数据类型和数组的大小，类似temp = Array('i', 5)。对于数据类型有下面的表格对应：
>‘c’: ctypes.c_char, ‘u’: ctypes.c_wchar,
>‘b’: ctypes.c_byte, ‘B’: ctypes.c_ubyte,
>‘h’: ctypes.c_short, ‘H’: ctypes.c_ushort,
>‘i’: ctypes.c_int, ‘I’: ctypes.c_uint,
>‘l’: ctypes.c_long, ‘L’: ctypes.c_ulong,
>‘f’: ctypes.c_float, ‘d’: ctypes.c_double

### 2.1.2 使用Manager共享数据
```
from multiprocessing import Process, Manager
 
def Foo(i,dic):
    dic[i] = 100+i
    print(dic.values())
 
if __name__ == '__main__':
    manage = Manager()
    dic = manage.dict()
    for i in range(10):
        p = Process(target=Foo, args=(i,dic))
        p.start()
        p.join()
```
Manager比Array要好用一点，因为它可以同时保存多种类型的数据格式。

### 2.1.3 使用queues的Queue类共享数据
```
import multiprocessing
from multiprocessing import Process
from multiprocessing import queues

def foo(i, arg):
    arg.put(i)
    print('The Process is ', i, 'and the queue\'s size is ', arg.size())

if __name__ == '__main__':
    li = queues.Queue(20, ctx=multiprocrssing)
    for i in range(10):
        p = Process(target=foo, args=(i, li,))
        p.start()
```
这里就有点类似上面的队列了。从运行结果里，你还能发现数据共享中存在的脏数据问题。另外，比较悲催的是multiprocessing里还有一个Queue，一样能实现这个功能。

### 2.2 进程锁

为了防止和多线程一样的出现数据抢夺和脏数据的问题，同样需要设置进程锁。与threading类似，在multiprocessing里也有同名的锁类RLock, Lock, Event, Condition, Semaphore，连用法都是一样样的！

### 2.3 进程池
既然有线程池，那必然也有进程池。但是，python给我们内置了一个进程池，不需要像线程池那样需要自定义，你只需要简单的from multiprocessing import Pool。
```
from multiprocessing import Pool
import time
 
def f1(args):
    time.sleep(1)
    print(args)
 
if __name__ == '__main__':
    p = Pool(5)
    for i in range(30):
        p.apply_async(func=f1, args= (i,))
    p.close()           # 等子进程执行完毕后关闭进程池
    # time.sleep(2)
    # p.terminate()     # 立刻关闭进程池
    p.join()
```
进程池内部维护一个进程序列，当使用时，去进程池中获取一个进程，如果进程池序列中没有可供使用的进程，那么程序就会等待，直到进程池中有可用进程为止。
进程池中有以下几个主要方法：
* apply：从进程池里取一个进程并执行
* apply_async：apply的异步版本
* terminate:立刻关闭进程池
* join：主进程等待所有子进程执行完毕。必须在close或terminate之后。
* close：等待所有进程结束后，才关闭进程池。

## 三、协程
线程和进程的操作是由程序触发系统接口，最后的执行者是系统，它本质上是操作系统提供的功能。而协程的操作则是程序员指定的。

协程存在的意义：对于多线程应用，CPU通过切片的方式来切换线程间的执行，线程切换时需要耗时。协程，则只使用一个线程，分解一个线程成为多个“微线程”，在一个线程中规定某个代码块的执行顺序。

协程的适用场景：当程序中存在大量不需要CPU的操作时（IO）。

在不需要自己“造轮子”的年代，同样有第三方模块为我们提供了高效的协程，这里介绍一下greenlet和gevent。本质上，gevent是对greenlet的高级封装，因此一般用它就行，这是一个相当高效的模块。

在使用它们之前，需要先安装，可以通过源码，也可以通过pip。

### 3.1 greenlet
```
from greenlet import greenlet
 
def test1():
    print(12)
    gr2.switch()
    print(34)
    gr2.switch()
 
def test2():
    print(56)
    gr1.switch()
    print(78)
 
gr1 = greenlet(test1)
gr2 = greenlet(test2)
gr1.switch()
```
实际上，greenlet就是通过switch方法在不同的任务之间进行切换。

### 3.2 gevent
```
from gevent import monkey; monkey.patch_all()
import gevent
import requests
 
def f(url):
    print('GET: %s' % url)
    resp = requests.get(url)
    data = resp.text
    print('%d bytes received from %s.' % (len(data), url))
 
gevent.joinall([
        gevent.spawn(f, 'https://www.python.org/'),
        gevent.spawn(f, 'https://www.yahoo.com/'),
        gevent.spawn(f, 'https://github.com/'),
])
```
通过joinall将任务f和它的参数进行统一调度，实现单线程中的协程。代码封装层次很高，实际使用只需要了解它的几个主要方法即可。