---
title: 刷题之按序打印python3
date: 2019-10-18 11:21:57
tags:
    - python3
    - 多线程
    - 信号量
    - leetcode
categories: "coding"
---

### 废话
刚好折腾完自动部署,开始一波leetcode刷题之旅

### 题目---按序打印
[题目地址](https://leetcode-cn.com/problems/print-in-order/)

我们提供了一个类:
```java
public class Foo {
    public void one(){
        print("one")
    }
    public void two(){
            print("two")
        }
    public void three(){
            print("three")
        }
}
```
三个不同的线程将会共用一个 Foo 实例。
* 线程 A 将会调用 one() 方法
* 线程 B 将会调用 two() 方法
* 线程 C 将会调用 three() 方法

请设计修改程序，以确保 two() 方法在 one() 方法之后被执行，three() 方法在 two() 方法之后被执行。

### Examples
#### 示例1
输入: [1,2,3]  
输出: "onetwothree"  
解释:   
有三个线程会被异步启动。
输入 [1,2,3] 表示线程 A 将会调用 one() 方法，线程 B 将会调用 two() 方法，线程 C 将会调用 three() 方法。  
正确的输出是 "onetwothree"。

#### 示例2
输入: [1,3,2]  
输出: "onetwothree"
解释:   
输入 [1,3,2] 表示线程 A 将会调用 one() 方法，线程 B 将会调用 three() 方法，线程 C 将会调用 two() 方法。  
正确的输出是 "onetwothree"。

#### 注意
尽管输入中的数字似乎暗示了顺序，但是我们并不保证线程在操作系统中的调度顺序。

你看到的输入格式主要是为了确保测试的全面性。


#### 转载
来源：力扣（LeetCode）  
链接：https://leetcode-cn.com/problems/print-foobar-alternately  
著作权归领扣网络所有。商业转载请联系官方授权，非商业转载请注明出处。


### 解题
初始化代码
```python
class Foo:
    def __init__(self):
        pass


    def first(self, printFirst: 'Callable[[], None]') -> None:
        
        # printFirst() outputs "first". Do not change or remove this line.
        printFirst()


    def second(self, printSecond: 'Callable[[], None]') -> None:
        
        # printSecond() outputs "second". Do not change or remove this line.
        printSecond()


    def third(self, printThird: 'Callable[[], None]') -> None:
        
        # printThird() outputs "third". Do not change or remove this line.
        printThird()
```

####思路
要执行按序打印,取巧办法可以选择在second及third方法分别使用time.sleep()方法,让线程休眠,已完成按序执行操作。例如:
```python
def second(self, printSecond: 'Callable[[], None]') -> None:
    # printSecond() outputs "second". Do not change or remove this line.
    time.sleep(1)
    printSecond()

def third(self, printThird: 'Callable[[], None]') -> None:
    # printThird() outputs "third". Do not change or remove this line.
    time.sleep(2)
    printThird()
```
但是这种方法真的很无聊,也不严谨,故舍弃!!!(都靠取巧做题目,怎么提升水平呢....)  
如果学过操作系统这本书的话,应该能想到信号量这一个概念,而本题就可以使用信号量,来解决问题。

#### 什么是信号量
信号量semaphore 是用于控制进入数量的锁。有哪些应用场景呢，比如说在读写文件的时候，一般只能只有一个线程在写，而读可以有多个线程同时进行，如果需要限制同时读文件的线程个数，这时候就可以用到信号量了（如果用互斥锁，就是限制同一时刻只能有一个线程读取文件）。又比如在做爬虫的时候，有时候爬取速度太快了，会导致被网站禁止，所以这个时候就需要控制爬虫爬取网站的频率。

semaphore在python3的threading库中,使用只需
```python
from threading import Semaphore

vex = Semaphore(value=1) #即构造一个内部维护计数器大小为1的信号量
```
semaphore主要有两个方法:
```
每当调用acquire()时，内置计数器-1,直到为0的时候阻塞
每当调用release()时，内置计数器+1，并让某个线程的acquire()从阻塞变为不阻塞
计数器不能小于0，当计数器为0时，acquire()将阻塞线程直到其他线程调用release()。
```
那么,学会使用这个信号量,这道问题也就变得很简单了

只需要在类Foo初始化时构造三个信号量,并让方法去控制信号量的增减即可
```python
class Foo:
    def __init__(self):
        self.s1 = Semaphore(1) #初始化1,保证first()方法不阻塞,优先执行
        self.s2 = Semaphore(0) #初始化0,阻塞住two()方法,直到其计数器大于0
        self.s3 = Semaphore(0) #初始化0,阻塞住third()方法,直到其计数器大于0

    def first(self, printFirst: 'Callable[[], None]') -> None:
        
        # printFirst() outputs "first". Do not change or remove this line.
        self.s1.acquire() #此时s1 > 0 可以执行
        printFirst()
        self.s2.release() # s2 + 1


    def second(self, printSecond: 'Callable[[], None]') -> None:
        
        # printSecond() outputs "second". Do not change or remove this line.
        self.s2.acquire() #在类刚开始执行时,计数器为0,只有当first()执行成功, s2 + 1才会从堵塞状态进入运行状态
        printSecond()
        self.s3.release() # 同理


    def third(self, printThird: 'Callable[[], None]') -> None:
        
        # printThird() outputs "third". Do not change or remove this line.
        self.s3.acquire() # 同理
        printThird()
```

### 效果
执行结果：通过  
执行用时 :
72 ms, 在所有 python3 提交中击败了93.37%的用户  
内存消耗 :16.1 MB, 在所有 python3 提交中击败了100.00%的用户

![avatar](/images/print-in-order.png)
