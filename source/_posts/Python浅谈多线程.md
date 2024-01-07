---
title: Python浅谈多线程
categories: 开发
keywords: 多线程
top_img: /img/104512.webp
cover: /img/3656799.webp
abbrlink: 6e07425f
date: 2022-02-22 23:46:21
---

##  前言：

只要玩过**爬虫**的，就知道线程的必要性。但是我学习线程的路子比较野～

所以学的不是那么**系统**，最近没事来看看把重要的部分都来掌握下。

然后也就“**简简单单**”水一篇重点部分（主要我怕后面忘记了QAQ）

##  引入：

`什么是GIL?`  在实现Python解析器(CPython)时所引入的一个概念，GIL这把超级大锁，是加在全局上的

同一个时刻只有一个线程在一个cpu上执行字节码, 无法将多个线程映射到多个cpu上执行

但是GIL会主动释放：1. 根据执行的字节码行数以及时间片释放；2. 在遇到io的操作时候主动释放

> *In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)*

看个例子：这个怎么都不会实现为0。具体如何解决看后面

```python
import threading

total = 0
def add():
    global total
    for i in range(1000000):
        total += 1
def desc():
    global total
    for i in range(1000000):
        total -= 1

thread1 = threading.Thread(target=add)
thread2 = threading.Thread(target=desc)
thread1.start()
thread2.start()
print(total)
```

## Thread的基础

这里面的几个点：

1. 除了最基础的写法，还可以继承`threading.Thread`重构方法：例如Run方法，将逻辑写在里面
2. 使用多线程的基本API的掌握，这里就不贴每个API什么意思。翻翻文档就好啦

```python
import time
import threading

class GetDetailHtml(threading.Thread):
    def __init__(self, name):
        super().__init__(name=name)

    def run(self):
        print("get detail html started")
        time.sleep(2)
        print("get detail html end")

class GetDetailUrl(threading.Thread):
    def __init__(self, name):
        super().__init__(name=name)

    def run(self):
        print("get detail url started")
        time.sleep(4)
        print("get detail url end")

if __name__ == "__main__":
    thread1 = GetDetailHtml("get_detail_html")
    thread2 = GetDetailUrl("get_detail_url")
    start_time = time.time()
    thread1.start()
    thread2.start()
    thread1.join()
    thread2.join()
    # 当主线程退出的时候， 子线程kill掉
    print("last time: {}".format(time.time() - start_time))
```

## 多个线程之间的变量

> 会产生需求就是，大家同时操作一个变量。这个变量可以穿梭在各个线程之间

当然这里有个很大的BUG，我当时就很疑惑🤔.

就是加入现在有2个文件，其中一个是`utils.py`文件中的一个变量test

如果你是 `import utils`然后使用的是`utils.test`这个的话是可以的

但是你是`from utils import test`使用`test`的话，不管你怎么改变，他就算一个定值了

我当时就很疑惑为什么`flag`变量无法是`while`退出，现在知道避坑了。

说了这么多，其实就是2个方式

1. 使用`gloabl全局变量`
2. 使用`queue`（推荐）,具体的API翻翻文档/源码都行，不多赘述

## 花里胡哨的锁

###  Lock、RLock

> 使用锁就很好解决了上面的问题，就可以得到答案为0

`Lock`与`RLock`的唯一区别就是有效减少死锁的发生

他们都有共同的`.acquire()`与`.release()`这个两个方式

但是`lock.acquire()`在不经意之间使用了2次，则会互相等待发生死锁，都不会动了

而`RLock.acquire()`可以使用多次并且多次释放，减少死锁竞争的情况。

###  condition 使用

> 这个使用的情况是：假设现在有2个线程，线程a执行完毕之后呢线程b再执行，线程b执行完毕之后呢再执行线程a
>
> 互相通知对方，多线执行，但可以实现交替

特别注意： 启动的顺序很重要‼️

掌握2个API： `.notify()`与`.wait()`顾名思义：一个通知一个等待

`通知`就是通知等待的对方你可以开始执行了，`等待`就是等待对方来通知我

###  Semaphore 使用

> 这个使用的情况就是：for循环的话一下子启动20个循环，设备顶不住。
>
> 使用Semaphore就可以决定一次性开启多少个

课上这个例子比较经典：

注意`.release()`与`sem.acquire()`的位置

```python
import threading
import time

class HtmlSpider(threading.Thread):
    def __init__(self, url, sem):
        super().__init__()
        self.url = url
        self.sem = sem

    def run(self):
        time.sleep(2)
        print("got html text success")
        self.sem.release()

class UrlProducer(threading.Thread):
    def __init__(self, sem):
        super().__init__()
        self.sem = sem

    def run(self):
        for i in range(20):
            self.sem.acquire()
            html_thread = HtmlSpider("https://baidu.com/{}".format(i), self.sem)
            html_thread.start()

if __name__ == "__main__":
    sem = threading.Semaphore(3)
    url_producer = UrlProducer(sem)
    url_producer.start()
```

## 线程池的使用

> 线程池相比较于多线程，更加具有智能化，说白了就是更加省心

为什么要线程池 ? 

1. 主线程中可以获取某一个线程的状态或者某一个任务的状态，以及返回值
2. 当一个线程完成的时候我们主线程能立即知道
3. `from concurrent.futures import Future`的futures和多进程编码接口一致

最好自己改一下使用with上下文`with ThreadPoolExecutor(3) as executor:`：

```python
# 一个个提交
import time
from concurrent.futures import ThreadPoolExecutor

def get_html(times):
    print("start sleep {}".format(times))
    time.sleep(times)
    print("get page {} success".format(times))
    return times

executor = ThreadPoolExecutor(max_workers=1)
task = executor.submit(get_html, 3)  # submit 是立即返回
task2 = executor.submit(get_html, 2)  # submit 是立即返回
print(task.done())  # 执行完成没
print(task2.cancel())  # 因为总的线程为1 task2没有执行 所以可以取消
time.sleep(4)
print(task.done())
print(task.result())

# 批量提交
import time
from concurrent.futures import ThreadPoolExecutor, as_completed, wait
def get_html(times):
    print("start sleep {}".format(times))
    time.sleep(times)
    print("get page {} success".format(times))
    return times

oneList = [2, 4, 3]
executor = ThreadPoolExecutor(max_workers=2)
# allTask = [executor.submit(get_html, each) for each in oneList]
# wait(allTask)  # 所有线程执行完毕再走
# # 一旦有完成的了就能获取到(谁先完成)
# for future in as_completed(allTask):  # 获取到已经完成的了
#     data = future.result()
#     print("! get {}".format(str(data)))
# 换个写法 (但是会按照oneList的顺序）
for data in executor.map(get_html, oneList):
    print("! get {}".format(str(data)))
```

## 多进程编程

> - 耗cpu的操作，用多进程编程， 
>
> - 对于io操作来说， 使用多线程编程
> - 进程切换代价要高于线程

一般都是用多线程，多进程与多线程的库使用差不多～

```python
import time
from concurrent.futures import ProcessPoolExecutor, as_completed

def random_sleep(n):
    time.sleep(n)
    return n

if __name__ == "__main__":
    with ProcessPoolExecutor(3) as executor:
        all_task = [executor.submit(random_sleep, (num)) for num in [2] * 30]
        start_time = time.time()
        for future in as_completed(all_task):
            data = future.result()
            print("exe result: {}".format(data))
        print("last time is: {}".format(time.time() - start_time))
```

----

1. 共享全局变量在多进程中是不使用的
2. multiprocessing中的queue不能用于pool进程池
3. pool中的进程间通信需要使用manager中的queue
4. 使用Pipe通信，但是只能适用于2个进程

```python
from queue import Queue # 多进程不能用

from multiprocessing import Queue # 正常的多进程使用

from multiprocessing import Manager # pool里面使用
Manager().Queue() 

from multiprocessing import Pipe
```



