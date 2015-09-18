title: java多线程基础
date: 2013-09-24 11:22
categories: java 
---
以前看过一段时间java多线程，很久没用到，忘记了。最近又遇到了一些多线程的问题，重新查了些资料，再提炼一下。本文不涉及java.util.concurrent包里的API 
<!--more-->

# 线程的状态 

线程的运行状态主要有runnable、running、waiting、timed_waiting、blocked等。主要有2类API，可以控制线程的运行状态 

第一类是调度类，不涉及object monitor和synchronized方法，这类API包括yield()、sleep()、join()等 

第二类是同步类，涉及到object monitor和synchronized方法，这类API包括wait()、notify()、notifyAll() 

总的来说，第一类API比第二类要简单 

# 调度类API 

这类API不涉及同步方法，是直接控制线程的运行状态，都是Thread类定义的方法 

yield()会使线程从running状态切换到runnable状态，但是也有可能JVM立刻又把该线程切换回running状态（优先级高、无其他线程等原因），可以说，这个方法只是“建议”jvm先去执行其他线程，运行结果是不能保证的 

sleep()需要跟一个参数，单位是毫秒，会使线程进入timed_waiting状态，sleep时间结束以后，会回到runnable状态 

join()使当前线程进入waiting状态，等join的线程执行完毕之后，才回到runnable状态。举例来说，有2个线程t1和t2，在t1里执行了t2.join()，则t1进入waiting状态，等t2执行完毕之后，t1才回到runnable状态
```
public static void main(String[] args) throws InterruptedException {

		final Thread thread = new Thread(new Runnable() {

			@Override
			public void run() {
				try {
					Thread.sleep(10000);// 此行代码使该线程进入timed_waiting状态
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

		});

		thread.start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				try {
					thread.join();// 此行代码使该线程进入waiting状态，等待thread执行完毕
				} catch (InterruptedException e) {
					e.printStackTrace();
				}
			}

		}).start();

	}
```
用jstack看到这2个线程的状态： 
![](http://dl.iteye.com/upload/attachment/0081/9224/f9a71e7d-2872-3f27-8eff-6252e27b4a6e.png)

Thread-0由于执行了sleep()方法，而进入timed_waiting状态；Thread-1由于执行了join()方法，而进入waiting状态 

调度类的API不涉及synchronized方法，所以也不涉及到blocked状态 

# synchronized关键字与object monitor 

同步的关键字是synchronized，其实我觉得叫“锁定”会比较好，不知道怎么理解“同步”这个词 

synchronized方法包括2种，一种是标注了synchronized关键字的方法，一种是synchronized代码块 

本质上差不多，只要是同步方法，都有一个object monitor：对于synchronized关键字方法来说，monitor就是该方法所属的类的实例；对于synchronized代码块来说，monitor是在synchronized关键字后面直接指定的 

当线程执行到同步方法或者同步代码块的时候，就会获取object monitor，并继续执行；但是，同一时刻，只有唯一一个线程可以获取object monitor，其他的线程就会阻塞，进入blocked状态，等待持有object monitor的那个线程释放object monitor 

这就是同步代码的基本状况，如果没有同步类的API的话，多线程执行就无法控制了，只能线程A获取->线程A释放->线程B获取->线程B释放->线程C获取……，这样顺序执行下去，所以就有了下面的同步类API 

# 同步类API 

前面说到了，同步的代码（方法或代码块），都有一个object monitor，可以通过在object monitor上执行wait()、notify()、notifyAll()方法，来进行多线程之间的通信 

在object monitor上执行wait()方法，会使当前的线程释放锁，并进入waiting状态；并将该线程加入object monitor内部的“等待线程列表”中 

在object monitor上执行notify()方法，会从“等待线程列表”中随机挑选一个线程唤醒，进入runnable状态（如果没有获取object monitor，则会立刻进入blocked状态） 

在object monitor上执行notifyAll()方法，会将“等待线程列表”中所有的线程都唤醒，全部进入runnable状态（但是随后没有获取object monitor的线程，会立刻进入blocked状态） 

注意，当一个线程调用了object.wait()方法进入waiting状态之后，如果没有其他线程帮它调用object.notify()方法，则该线程将永远停留在waiting状态，此线程永不结束 

贴一段示例代码进行说明：
```
public static void main(String[] args) {

		final Object o = new Object();

		new Thread(new Runnable() {

			@Override
			public void run() {
				synchronized (o) {
					try {
						System.out.println("me first");// 这个线程先执行，是期望的结果，但是不能保证
						Thread.sleep(15000);
						o.wait();
						System.out.println("wake up and done");
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}

		}).start();

		new Thread(new Runnable() {

			@Override
			public void run() {
				synchronized (o) {
					try {
						System.out.println("it's my time");// 如果这个线程先执行，则另一个线程将永远waiting
						Thread.sleep(15000);
						o.notify();
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}
			}

		}).start();

	}
```


阶段1：thread-0进入timed_waiting状态，thread-1没有获取到object monitor，进入blocked状态 
![](http://dl.iteye.com/upload/attachment/0081/9246/617979d4-2de2-3aab-b773-e28f38a7e579.png)

控制台输出：me first 

阶段2：thread-0调用了o.wait()方法，释放锁并进入waiting状态；thread-1执行sleep()方法，进入timed_waiting状态 
![](http://dl.iteye.com/upload/attachment/0081/9250/61d015af-f27a-3956-a90f-745866724349.png)

控制台输出：it's my time 

阶段3：thread-1执行o.notify()方法，唤醒thread-0，自己运行结束，释放锁（并不是notify方法释放了锁）；thread-0执行最后一行代码，然后也结束；两个线程都执行完毕 

控制台输出：wake up and done 

但是，这里有很强的随机性，每次执行的效果是不同的！如果是thread-1先执行的话，那么当thread-0调用o.wait()之后，就会永远停留在waiting状态，该线程永远不会结束 

# 同步类API的限制 

上述3个方法，都是只能在synchronized代码里调用的，否则就会报错。准确的说，虽然这3个方法都是Object类定义的方法，但是只有成为object monitor的实例才能调用 

比如以下这段代码是不能执行的，o是object monitor，而o1不是，所以不能调用o1.wait()：
```
public static void main(String[] args) {

		final Object o = new Object();

		final Object o1 = new Object();

		new Thread(new Runnable() {

			@Override
			public void run() {

				synchronized (o) {
					try {
						o1.wait();// 错误
					} catch (InterruptedException e) {
						e.printStackTrace();
					}
				}

			}

		}).start();

	}
```
# 总结 

yield()、sleep()、join()都是Thread类定义的方法，不涉及同步方法，因此也不会使线程切换到blocked状态，也不涉及object monitor的释放 

wait()、notify()、notifyAll()是Object类定义的方法，但是只能在object monitor实例上调用 

wait()方法会立刻释放object monitor，并使当前线程进入waiting状态 

notify()和notifyAll()方法只是唤醒此前调用了wait()方法的线程，但是自己并不会释放锁 
![](http://dl.iteye.com/upload/attachment/0081/9306/d8877b2f-80fe-3e74-8344-e0a57cc16c41.png)
本文也参考了：
[http://dylanxu.iteye.com/blog/1322066](http://dylanxu.iteye.com/blog/1322066) 
[http://www.cnblogs.com/adamzuocy/archive/2010/03/08/1680851.html](http://www.cnblogs.com/adamzuocy/archive/2010/03/08/1680851.html)