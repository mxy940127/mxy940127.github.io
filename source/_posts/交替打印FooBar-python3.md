---
title: 交替打印FooBar(python3)
date: 2019-10-18 14:34:46
tags:
    - python3
    - 多线程
    - 信号量
    - leetcode
categories: "coding"
---

### 题目---交替打印FooBar
[题目地址](https://leetcode-cn.com/problems/print-foobar-alternately/)

我们提供了一个类:
```java
public class FooBar {
    public void foo() {
        for (int i = 0; i < n; i++) {
          print("foo");
        }
    }

    public void bar() {
        for (int i = 0; i < n; i++) {
          print("bar");
        }
    }
}

```
两个不同的线程将会共用一个 FooBar 实例。其中一个线程将会调用 foo() 方法，另一个线程将会调用 bar() 方法。

请设计修改程序，以确保 "foobar" 被输出 n 次


### Examples
#### 示例1
输入: n = 1  
输出: "foobar"  
解释: 这里有两个线程被异步启动。其中一个调用 foo() 方法, 另一个调用 bar() 方法，"foobar" 将被输出一次。


#### 示例2
输入: n = 2  
输出: "foobarfoobar"  
解释: "foobar" 将被输出两次。

#### 转载
来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/print-foobar-alternately  
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。

### 解题
初始化代码
```python
class FooBar:
    def __init__(self, n):
        self.n = n


    def foo(self, printFoo: 'Callable[[], None]') -> None:
        
        for i in range(self.n):
            
            # printFoo() outputs "foo". Do not change or remove this line.
        	printFoo()


    def bar(self, printBar: 'Callable[[], None]') -> None:
        
        for i in range(self.n):
            
            # printBar() outputs "bar". Do not change or remove this line.
        	printBar()
```

####思路
要执行交替打印,有了之前顺序打印的基础,事情变得很简单。

创建两个信号量A,B---A初始化1,B初始化0。A执行完B加1,B执行完A加1即可,因此代码如下


```python
from threading import Semaphore
class FooBar:
    def __init__(self, n):
        self.n = n
        self.s1 = Semaphore(1)
        self.s2 = Semaphore(0)

    def foo(self, printFoo: 'Callable[[], None]') -> None:
        
        for i in range(self.n):
            
            # printFoo() outputs "foo". Do not change or remove this line.
            self.s1.acquire()
            printFoo()
            self.s2.release()


    def bar(self, printBar: 'Callable[[], None]') -> None:
        
        for i in range(self.n):
            
            # printBar() outputs "bar". Do not change or remove this line.
            self.s2.acquire()
            printBar()
            self.s1.release()
```

### 效果
执行结果：通过  
执行用时 :104 ms, 在所有 python3 提交中击败了85.40%的用户  
内存消耗 :16.1 MB, 在所有 python3 提交中击败了100.00%的用户  

![avatar](/交替打印FooBar-python3/print-foobar-alternately.png)
