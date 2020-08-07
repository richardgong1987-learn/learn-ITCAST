# 并发编程-上部分

## 3. Java 线程

### 3.1 创建和运行线程

#### 方法一，直接使用 Thread

```java
// 创建线程对象
Thread t = new Thread() {
public void run() {
// 要执行的任务
}
};
// 启动线程
t.start();
```

#### 方法二，使用 Runnable 配合 Thread

```java
Runnable runnable = new Runnable() {
public void run(){
// 要执行的任务
}
};
// 创建线程对象
Thread t = new Thread( runnable );
// 启动线程
t.start();
```

#### 方法三，FutureTask 配合 Thread

FutureTask 能够接收 Callable 类型的参数，用来处理有返回结果的情况

```java
// 创建任务对象
FutureTask<Integer> task3 = new FutureTask<>(() -> {
log.debug("hello");
return 100;
});
// 参数1 是任务对象; 参数2 是线程名字，推荐
new Thread(task3, "t3").start();
// 主线程阻塞，同步等待 task 执行完毕的结果
Integer result = task3.get();
log.debug("结果是:{}", result);
```



### 3.7 sleep 与 yield

#### **sleep特点**

1. 调用 sleep 会让当前线程从 Running 进入 Timed Waiting 状态（阻塞）

2. 其它线程可以使用 interrupt 方法打断正在睡眠的线程，这时 sleep 方法会抛出 InterruptedException

3. 睡眠结束后的线程未必会立刻得到执行 (它需要等待CPU时间片分配到给自己才会执行,也就是执行权)

4. 建议用 TimeUnit 的 sleep 代替 Thread 的 sleep 来获得更好的可读性



#### **yield**特点

1. 调用 yield 会让当前线程从 Running 进入 Runnable 就绪状态，然后调度执行其它线程
2. 具体的实现依赖于操作系统的任务调度器



#### sleep与yield的区别:

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

unpark



## 4. 共享模型之管程



### 临界区 Critical Section

- 一个程序运行多个线程本身是没有问题的

- 问题出在多个线程访问共享资源

  - 多个线程读共享资源其实也没有问题

  - 在多个线程对共享资源读写操作时发生指令交错，就会出现问题

- 一段代码块内如果存在对共享资源的多线程读写操作，称这段代码块为临界区





### 竞态条件 Race Condition

多个线程在临界区内执行，由于代码的执行序列不同而导致结果无法预测，称之为发生了竞态条件



### 4.2 synchronized 解决方案

* 应用之互斥

为了避免临界区的竞态条件发生，有多种手段可以达到目的。
阻塞式的解决方案：synchronized，Lock

非阻塞式的解决方案：原子变量



### 4.6 Monitor 概念



### 4.8 wait notify 的正确姿势

#### sleep(long n) 和  wait(long n) 的区别

1) sleep 是 Thread 方法，而 wait 是 Object 的方法 

2) sleep 不需要强制和 synchronized 配合使用，但 wait 需要和 synchronized 一起用 

3) sleep 在睡眠的同时，不会释放对象锁的，但 wait 在等待的时候会释放对象锁

 4) 它们状态 TIMED_WAITING



####  模式之保护性暂停

```java
ynchronized(lock) {
while(条件不成立) {
lock.wait();
}
// 干活
}
```

####  

### 4.9 Park & Unpark

#### 模式之生产者消费者

#### Park & Unpark与 Object 的 wait & notify 相比

- ##### wait，notify 和 notifyAll 必须配合 Object Monitor 一起使用，而 park，unpark 不必

- park & unpark 是以线程为单位来【阻塞】和【唤醒】线程，而 notify 只能随机唤醒一个等待线程，notifyAll
  是唤醒所有等待线程，就不那么【精确】

- park & unpark 可以先 unpark，而 wait & notify 不能先 notify



### 4.13 ReentrantLock

相对于 synchronized 它具备如下特点

- 可中断
- 可以设置超时时间
- 可以设置为公平锁
- 支持多个条件变量





## 5. 共享模型之内存