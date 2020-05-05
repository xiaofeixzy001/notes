[TOC]

# 简介

3.2版本引入的模块

异步并行任务编程模块，提供一个高级的异步可执行的遍利接口。

futures模块提供了2个池执行器:

ThreadPoolExecutor异步调用的==线程池==的Executor

ProcessPoolExecutor异步调用的==进程池==的Executor

# ThreadPoolExecutor

## 创建线程池

ThreadPoolExecutor(max_workers=1)

创建一个线程池,池中至多创建max_workers个线程池来同时异步执行，返回池执行器对象。

```python
from concurrent.futures import ThreadPoolExecutor

executor = ThreadPoolExecutor(max_workers=3)
print(executor)
```



## 提交任务

submit(fn, *args, **kwargs)

提交任务(执行的函数及其参数)，返回Future实例

```python
def worker(x, y):
    return x+y

fs = executor.submit(worker, x, y)
executor.shutdown(wait=True)
print(fs)
```

shutdown(wait=True): 清理池

## 任务管理

done() 如果调用被成功的取消或执行完成，返回True

cancelled() 如果调用被成功的取消，返回True

running() 如果正在运行且不能被取消，返回True

cancel() 尝试取消调用。如果已经执行且不能取消返回False，否则返回True

result(timeout=None) 取返回的结果，timeout为None，一直等待返回；timeout设置到期，抛出concurrent.futures.TimeoutError异常

exception(timeout=None) 取返回的异常，timeout为None，一直等待返回；timeout设置到期，抛出concurrent.futures.TimeoutError异常

## 示例

异步，进程池和线程池方便切换

```python
import threading
from concurrent import futures
import logging
import time

# 输出格式定义
FORMAT = "%(asctime)-15s\t [%(processName)s:%(threadName)s, %(process)d:%(thread)8d] %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

def worker(n):
    logging.info('Begin to work {}'.format(n))
    time.sleep(5)
    logging.info('finished {}'.format(n))

if __name__ == '__main__':

    # 创建线程池,池内线程数最大为3
    executor = futures.ThreadPoolExecutor(max_workers=3)

    # 创建进程池,池的容量为3
    # executor = futures.ProcessPoolExecutor(max_workers=3)

    fs = []
    for i in range(3):  # 提交3个任务
        f1 = executor.submit(worker, i)
        fs.append(f1)

    for i in range(3, 6):  # 再次提交3个任务,是否会阻塞? 不阻塞
        f2 = executor.submit(worker, i)
        print('~~~阻塞~~~')
        fs.append(f2)

    while True:
        time.sleep(2)
        logging.info(threading.enumerate())  # 查看当前线程池有几个线程

        flag = True
        for f in fs:
            logging.info(f.done())  # 是否还有未完成的任务,完成True,未完成False
            # flag = flag and f.done()
            flag = f.done()

        print("*" * 30)

        if flag:  # 如果所有任务执行完毕,清理池,池中线程全部杀掉
            executor.shutdown()
            logging.info(threading.enumerate())
            break
```

线程池一旦创建了线程，就不需要频繁清除。

## 示例2

```python
import random
import time
import threading
from concurrent.futures import ThreadPoolExecutor, ProcessPoolExecutor, as_completed


futures = {}


def test_func(s, key):
    print('enter~~~{}s, key={}'.format(threading.current_thread(), s, key))
    threading.Event().wait(s)
    if key == 3:
        raise Exception('{} failde'.format(key))
    return 'OK {}'.format(threading.current_thread())


def run(fs):
    while True:
        time.sleep(1)
        print(fs)
        print("*" * 20)
        # print(as_completed(fs), '*************')
        """
        只要有一个任务没有完成就阻塞,完成一个,执行一个.
        如果内部有异常result()会抛出异常
        有异常也算执行完了complete
        fs为空也不阻塞
        as_completed(fs): <generator object as_completed at 0x000001B23DD8E308>
        """
        for future in as_completed(fs):
            print(as_completed(fs), '1111111')
            print(future, '22222222')
            id = fs[future]
            try:
                print(id, future.result(), '333333333')
            except Exception as e:
                print(e)
                print(id, 'failed')

threading.Thread(target=run, args=(futures,)).start()

time.sleep(5)

with ThreadPoolExecutor(max_workers=3) as executor:
    for i in range(7):
        futures[executor.submit(test_func, random.randint(1, 8), i)] = i

```







# 上下文管理

支持上下文管理

concurrent.futures.ProcessPoolExecutor继承自concurrent.futures.base.Executor

而父类有\_\_enter\_\_、\_\_exit\_\_方法，支持上下文管理，可以使用with语句。

\_\_exit\_\_方法本质还是调用shutdown(wait=True)，就是一直阻塞到所有运行的任务完成。

```python
import threading
from concurrent import futures
import logging
import time

# 输出格式定义
FORMAT = "%(asctime)-15s\t [%(processName)s:%(threadName)s, %(process)d:%(thread)8d] %(message)s"
logging.basicConfig(format=FORMAT, level=logging.INFO)

def worker(n):
    logging.info('Begin to work {}'.format(n))
    time.sleep(5)
    logging.info('finished {}'.format(n))
    return n + 100

if __name__ == '__main__':

    # 创建线程池,池的容量为3
    # executor = futures.ThreadPoolExecutor(max_workers=3)

    # 创建进程池,池的容量为3
    # executor = futures.ProcessPoolExecutor(max_workers=3)
    with futures.ProcessPoolExecutor(max_workers=3) as executor:
        fs = []
        for i in range(3):  # 提交3个任务
            f1 = executor.submit(worker, i)
            fs.append(f1)

        for i in range(3, 6):  # 再次提交3个任务,是否会阻塞? 不阻塞
            f2 = executor.submit(worker, i)
            print('~~~阻塞~~~')
            fs.append(f2)

        while True:
            time.sleep(2)
            logging.info(threading.enumerate())  # 查看当前线程池有几个线程

            flag = True
            for f in fs:
                logging.info(f.done())  # 是否还有未完成的任务,完成True,未完成False
                # flag = flag and f.done()
                flag = f.done()
                if f.done():  # 如果做完了看看结果
                    logging.info("result = {}".format(f.result()))
            if flag: break
    logging.info("=== end ===")
    logging.info(threading.enumerate())  # 多进程时看主线程已经没有必要
```









# 总结

该库统一了线程池、进程池调用，简化了编程。

是Python简单的思想哲学的提现。

唯一缺点：无法设置线程名称，但这都不值一提。



