一个线程由一个方法启动，但是一个线程不等价于一个方法，一个线程里可能有多个方法。见如下代码：
```java

class TestThread extends Thread {

	public TestThread(String name) {
		super(name);
	}

	@Override
	public void run() {
		System.out.println("启动一个字线程，线程名为： " + Thread.currentThread().getName());
		runSub();
	}

	public void runSub() {
		System.out.println("在run方法里面被调用，线程名为： " + Thread.currentThread().getName());
		runSubToSub();
	}

	public void runSubToSub() {
		System.out.println("在runSub方法里面被调用，线程名为： " + Thread.currentThread().getName());
	}

	public static void main(String[] args) {
		System.out.println("当前主线程名为： " + Thread.currentThread().getName());

		mainSub();

		new TestThread("我是子线程01").start();
	}

	public static void mainSub() {
		System.out.println("在住线程中被调用，线程名为： " + Thread.currentThread().getName());
	}
}
```

其执行结果如下：
```markdown
当前主线程名为： main
在住线程中被调用，线程名为： main
启动一个字线程，线程名为： 我是子线程01
在run方法里面被调用，线程名为： 我是子线程01
在runSub方法里面被调用，线程名为： 我是子线程01
```

http请求，到业务处理，再到响应的过程，都是在一个线程里面的。
对tomcat来说，每一个进来的请求(request)都需要一个线程，直到该请求结束。tomcat会维护一个线程池，每一个http请求，会从线程池中取出一个空闲线程。默认初始化75个线程，可以进行修改。
