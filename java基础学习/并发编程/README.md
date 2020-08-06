# 并发编程

##  [自定义线程池 demo](custom_threadpool.md) 



## 3.Java 线程

### 3.7 sleep 与 yield



#### **sleep**

1. 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）

2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException

3. 睡眠结束后的线程未必会立刻得到执行 (它需要等待CPU时间片分配到给自己才会执行,也就是执行权)

4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性



#### **yield**

1. 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程
2. 具体的实现依赖于操作系统的任务调度器



sleep与yield的区别:

- 相同点:

  - 都是让出cpu使用权

- 不同点:

  - sleep会把当前线程状态进入Timed_waiting,这时CPU是不会分配时间片给状态为Timed_waiting的线程的

  - yield是把当前线程进入Runnable就绪状态,而这个Runnable状态仍然可以获取cpu使用权,cpu仍然会给它分配时间片

  - sleep有等待时间,时间结束后sleep才改成Runnable状态,cpu才会考虑分配时间片给sleep,yield没有等待时间直接多running进行Runnable状态

  - 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException, yield不会

    

#### **线程优先级**

- 线程优先级会提示（hint）调度器优先调度该线程，但它仅仅是一个提示，调度器可以忽略它  (不靠谱的东西)
- 如果 cpu 比较忙，那么优先级高的线程会获得更多的时间片，但 cpu 闲时，优先级几乎没作用



### 3.8 join 方法详解

join底层是用wait实现的,通过不断的判断isAlive来判断是否线程还在,如果还在就wait,这就是join的实现原理

一句话说就是:wait和join本质都是wait



### 3.9 interrupt 方法详解

#### 打断 sleep，wait，join 的线程

这几个方法都会让线程进入阻塞状态

打断 sleep 的线程, 会清空打断状态，以 sleep 为例



#### 打断 park 线程




