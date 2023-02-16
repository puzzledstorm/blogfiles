---
title: "Python装饰器"
date: 2023-02-16T08:19:12Z
description: …
draft: false
hideToc: false
enableToc: true
enableTocContent: false
author: puzzled
authorEmoji: 🦄
tocFolding: false
tocPosition: inner
tocLevels: ["h2", "h3", "h4"]
tags:
- questions
- python
series:
- questions
categories:
- questions
image: images/feature1/8.jpg

## 装饰器是什么？
装饰器是可调用的对象，其参数是另一个函数（被装饰的函数）。
装饰器可能会处理被装饰的函数，然后把它返回，或者将其替换成另一个 函数或可调用对象。

## examples

```
import functools
import time
from threading import Thread


def timer(func):
    def wrapper(*args, **kwargs):
        start = time.time()
        result = func(*args, **kwargs)
        print(f"execute func {func.__name__}, params {args, kwargs}, time passed {time.time() - start}s")
        return result

    return wrapper


def retry(times=1, delay=0):
    def outer(func):
        @functools.wraps(func)
        def inner(*args, **kwargs):
            t = times
            while t:
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    t -= 1
                    time.sleep(delay)
                    print(f"retry times: {times - t}, error: {e}, delay: {delay}")
                    if not t:
                        raise e

        return inner

    return outer


def log(text="no log output"):
    def decorator(func):
        @functools.wraps(func)
        def wrapper(*args, **kwargs):
            print(f"{text}, {func.__name__}")
            return func(*args, **kwargs)

        return wrapper

    return decorator


def async_func(f):
    def wrapper(*args, **kwargs):
        thread = Thread(target=f, args=args, kwargs=kwargs)
        thread.setDaemon(True)
        thread.start()
        return thread

    return wrapper


def main():
    @timer
    @log("this is a log")
    def do_something(t):
        time.sleep(t)
        return t

    @retry(times=3)
    def division_by_zero():
        return 1 / 0

    print(do_something(0.5))
    division_by_zero()


@timer
def test_async_func():
    @async_func
    def A():
        print("现在在执行A函数")
        print('A函数睡眠3秒钟')
        time.sleep(3)
        print("A函数执行完毕")
        return

    def B():
        print("现在在执行B函数")

    A()
    B()


if __name__ == '__main__':
    test_async_func()
    print("-"*156)
    main()


```

## references
```
https://www.liaoxuefeng.com/wiki/1016959663602400/1017451662295584
https://www.zywvvd.com/notes/coding/python/asyncio/threading/
https://www.zywvvd.com/notes/coding/python/fluent-python/chapter-7/python-decorate/python-decorate/
```
