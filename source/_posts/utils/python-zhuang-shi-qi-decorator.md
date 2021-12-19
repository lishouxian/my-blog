---
title: 'Python装饰器(Decorator)'
date: 2020-06-13 15:15:19
tags: [python]
categories: utils
published: true
hideInList: false
feature: 
isTop: false
---


Python 中的装饰器简单来说就是一种有特殊功能的函数，使用装饰器可以让代码更加简洁的同时也会让代码具有更高的阅读性。

话不多说直接上代码：

<!-- more -->

```Python
import time

def display_time(func):
  def wrapper():
    t1 = time.time()
    func()
    t2 = time.time()
    print(t2 - t1)
  return wrapper


def is_prime(num):
  if num < 2:
    return False
  elif num == 2:
    return True
  else:
    for i in range(2,num):
      
  

```

