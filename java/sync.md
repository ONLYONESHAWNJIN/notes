
1. sync锁的是对象，实际是对象头
2. 如果锁在方法上，那么锁的this对象
3. sync最好不要用常量，例如string，会造成潜在的死锁问题
4. sync锁住的代码块尽量小一点，让其他线程等待时间短。


---

1. 同步方法和非同步方法能同时调用吗？   -- 能同时

2. 脏读问题
 
	- 读的数据不是最新的数据	

3. sync支持可重入，

	当一个线程获取到锁后，如果在遇到sync代码时，不需要再次上锁就可以执行。

4. 线程状态：

	Java语言中定义了5种线程状态，在任意一个时间点，一个线程只能有且只有其中一种状态，这5种状态是：
	
	- 新建（New）
	- 运行（Runable）
	- 无限期等待（Waiting）
	- 限期等待（Timed Waiting）
	- 阻塞（Blocked）
	- 结束（Terminated）



5. 如果在sync抛出异常时会自动释放锁


	释放锁的3中情况：

	1. 同步代码块执行完
	2. 在同步代码块中抛出异常。
	3. 在代码块中执行wait()方法，线程进入等待队列，并释放锁。

6. volatile关键字

	-  防止指令重排
	-  内存在线程间的可见性
	-  并没有保证原子性

7. sync关键字

	- 可以保证原子性

8. 主线程，用户线程（非守护线程），守护线程

	**主线程**：main，但不是守护线程。
	
	**守护线程**：是指在程序运行的时候在后台提供一种通用服务的线程。如gc。
	
	**非守护线程**：也叫用户线程，由用户创建。
	
	**关系**：主线程和守护线程一起销毁，主线程和非守护线程互不影响。