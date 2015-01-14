title: 使用java闭锁实现并发测试
date: 2015-01-14 15:30:16 
---
最近看到公司的一个同事写了一个程序对我们的服务器进行并发测试，看了他代码令我很抓狂，
他用一个for循环，然后分别启动线程进行就搞定，类似这样的写法：
for(int i=0;i<5000;i++){
	Thread thread = new MyThread();
	thread.start();
}

上面的写法其实不是真正的并发测试，没有实现对5000个线程进行同步，让它们进行并发启动.因为之前看了《Java并发编程实战》，
看了里面对一种同步工具闭锁的介绍，利用这个类就可以实现真正的并发程序测试.

闭锁：一个同步辅助类，在完成一组正在其他线程中执行的操作之前，它允许一个或多个线程一直等待。
即，一组线程等待某一事件发生，事件没有发生前，所有线程将阻塞等待；
而事件发生后，所有线程将开始执行；
闭锁最初处于封闭状态，当事件发生后闭锁将被打开，一旦打开，闭锁将永远处于打开状态。

下面我写了一个类，利用闭锁（CountDownLatch）来实现对本网站并发进行测试.

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStream;
import java.io.InputStreamReader;
import java.net.URL;
import java.net.URLConnection;
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.CountDownLatch;

public class ThreadTest {  

    public static void main(String[] args) {  
        final int num = 10;  
        final CountDownLatch begin = new CountDownLatch(1);  
        final CountDownLatch end = new CountDownLatch(num);  

        for (int i = 0; i < num; i++) {  
            new Thread(new MyWorker(i, begin, end)).start();  
        }  

        // 睡眠十秒 
        try {  
            Thread.sleep(10000);  
        } catch (InterruptedException e1) {  
            e1.printStackTrace();  
        }  

        System.out.println("开始进行并发测试");  
        begin.countDown();  
        long startTime = System.currentTimeMillis();  

        try {  
            end.await();  
        } catch (InterruptedException e) {  
            e.printStackTrace();  
        } finally {  
            long endTime = System.currentTimeMillis();  
            System.out.println("结束并发测试 !");  
            System.out.println("花费时间: " + (endTime - startTime));  
        }  
    }  
}  

class MyWorker implements Runnable {  
    final CountDownLatch begin;  
    final CountDownLatch end;  
    final int id;  

    public MyWorker(final int id, final CountDownLatch begin,  
            final CountDownLatch end) {  
        this.id = id;  
        this.begin = begin;  
        this.end = end;  
    }  

    @Override  
    public void run() {  
        try {  
            System.out.println(this.id + " ready !");  
            begin.await();  
            // execute your logic
            URLConnection webUrlRequest = request("http://the090303.com");  
            List<String> readline = readLine(webUrlRequest.getInputStream());  

            System.out.println(readline);  

            Thread.sleep((long) (Math.random() * 10000));  
        } catch (Throwable e) {  
            e.printStackTrace();  
        } finally {  
            System.out.println(this.id + " 完成测试 !");  
            end.countDown();  
        }  
    }  

    public static URLConnection request(String requestUrl) throws IOException {  
        return new URL(requestUrl).openConnection();  
    }  

    public static List<String> readLine(InputStream inputStream) throws IOException {  
        List<String> list = new ArrayList<String>();  
        BufferedReader bufferReader = null;  
        try {  
            bufferReader = new BufferedReader(new InputStreamReader(inputStream));  

            String readline;  
            while ((readline = bufferReader.readLine()) != null) {  
                list.add(readline);  
            }  
        } finally {  
            bufferReader.close();  
        }  
        return list;  
    }
}

上面CountDownLatch是一种灵活的闭锁实现，闭锁状态包括一个计数器，该计数器被初始化一个正数，表示需要等待的事件数量。
调用countDown方法递减计算器，表示有一个事件已经发生了，而await方法要一直阻塞直到计数器为零，或者等待的线程中断、超时。
所以上面用两个闭锁来实现一个起始门（begin）跟结束门（end），起始门计数器初始值为1，结束门计算器初始值为线程数量，
每个线程就是在起始门等待，所以这样确保实现真正并发，而每个线程在最后做的一件事情是调用结束门（end）的countDown方法减1，
使得当所有的线程都执行完毕后，我们可以统计出所有线程对笔者网站进行访问所消耗的时间.

这里因为笔者安装了防ddos的脚本，只要同一个ip连接数达到150个就会被踢入黑名单，600秒后才能正常对网站进行访问，
所有将上面代码的num变量值改为200，说明我们要实现200个线程并发对笔者的网站 进行访问，这时程序后台就会出现异常，在本人的机子也ssh连接不了远程主机，说明使用了闭锁（CountDownLatch）
我们已经成功实现了对服务器进行并发测试，这真是一个不错的工具类.


import java.util.concurrent.Callable;
import java.util.concurrent.ExecutionException;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.Future;
import java.util.concurrent.FutureTask;

public class RunnableFutureTask {

	/**
	 * ExecutorService
	 */
	static ExecutorService mExecutor = Executors.newSingleThreadExecutor();

	/**
	 * @param args
	 */
	public static void main(String[] args) {
		runnableDemo();
		futureDemo();
	}

	/**
	 * runnable, 无返回值
	 */
	static void runnableDemo() {
		new Thread(new Runnable() {

			@Override
			public void run() {
				System.out.println("runnable demo : " + fibc(20));
			}
		}).start();
	}

	/**
	 * 其中Runnable实现的是void run()方法，无返回值；Callable实现的是 V
	 * call()方法，并且可以返回执行结果。其中Runnable可以提交给Thread来包装下
	 * ，直接启动一个线程来执行，而Callable则一般都是提交给ExecuteService来执行。
	 */
	static void futureDemo() {
		try {
			/**
			 * 提交runnable则没有返回值, future没有数据
			 */
			Future<?> result = mExecutor.submit(new Runnable() {

				@Override
				public void run() {
					fibc(20);
				}
			});

			System.out.println("future result from runnable : " + result.get());

			/**
			 * 提交Callable, 有返回值, future中能够获取返回值
			 */
			Future<Integer> result2 = mExecutor.submit(new Callable<Integer>() {
				@Override
				public Integer call() throws Exception {
					return fibc(20);
				}
			});

			System.out.println("future result from callable : " + result2.get());

			/**
			 * FutureTask则是一个RunnableFuture<V>，即实现了Runnbale又实现了Futrue<V>这两个接口，
			 * 另外它还可以包装Runnable(实际上会转换为Callable)和Callable
			 * <V>，所以一般来讲是一个符合体了，它可以通过Thread包装来直接执行，也可以提交给ExecuteService来执行
			 * ，并且还可以通过v get()返回执行结果，在线程体没有执行完成的时候，主线程一直阻塞等待，执行完则直接返回结果。
			 */
			FutureTask<Integer> futureTask = new FutureTask<Integer>(new Callable<Integer>() {
				@Override
				public Integer call() throws Exception {
					return fibc(20);
				}
			});
			// 提交futureTask
			mExecutor.submit(futureTask);
			System.out.println("future result from futureTask : " + futureTask.get());
			mExecutor.shutdown();
		} catch (InterruptedException e) {
			e.printStackTrace();
		} catch (ExecutionException e) {
			e.printStackTrace();
		}
	}

	/**
	 * 效率底下的斐波那契数列, 耗时的操作
	 * 
	 * @param num
	 * @return
	 */
	static int fibc(int num) {
		if (num == 0) {
			return 0;
		}
		if (num == 1) {
			return 1;
		}
		return fibc(num - 1) + fibc(num - 2);
	}
}
