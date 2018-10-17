# 测试ArrayList/ConcurrentHashMap等集合的安全性。
ArrayList为非线程安全集合，ConcurrentHashMap为线程安全集合。<br/>
100个线程，每个线程存入100个元素，ArrayList容易出现元素缺少，越界异常。ConcurrentHashMap比较正常；

```
package com.demo;

import java.util.ArrayList;
import java.util.List;
import java.util.Map;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.CountDownLatch;

public class ThreadTest {
	// 线程数量
	public static int TEST_THREAD_COUNT = 100;

	public static void main(String[] args) {
		try {
			// 线程数组
			Thread[] threads = new Thread[TEST_THREAD_COUNT];
			// 计数器
			final CountDownLatch latch = new CountDownLatch(TEST_THREAD_COUNT);
			// 创建并启动线程
			for (int i = 0; i < TEST_THREAD_COUNT; i++) {
				threads[i] = new Thread(new MyThread(latch));
				threads[i].start();
			}

			System.out.println("等待子线程执行完毕...");
			latch.await();
			System.out.println("子线程执行完毕...");
			// 打印集合数量
			System.out.println(MyThread.map.size());
			System.out.println(MyThread.list.size());
		} catch (InterruptedException e) {
			e.printStackTrace();
		}

	}
}

// 自定义线程类
class MyThread implements Runnable {
	// 非线程安全集合
	public static List<String> list = new ArrayList<>();
	// 线程安全集合
	public static Map<String, String> map = new ConcurrentHashMap<>();
	// 计数器
	private CountDownLatch latch;

	// 通过构造器传入构造器
	public MyThread(CountDownLatch latch) {
		this.latch = latch;
	}

	public MyThread() {

	}

	@Override
	public void run() {
		// 同步，ArrayList、HashMap等非同步集合使用同步代码块可保证安全
		synchronized ("sdf") {
			for (int i = 0; i < 100; i++) {
				// list.add(Thread.currentThread().getId()+"-"+i);
				map.put(Thread.currentThread().getId() + "-" + i, Thread.currentThread().getId() + "-" + i);
			}
		}
		// Vector、ConcurrentHashMap等线程安全集合不必使用同步代码块
		/*
		 * for (int i = 0; i <100; i++) {
		 * //list.add(Thread.currentThread().getId()+"-"+i);
		 * map.put(Thread.currentThread().getId()+"-"+i,
		 * Thread.currentThread().getId()+"-"+i); }
		 */
		// 计数器减1
		latch.countDown();
	}
}


```
