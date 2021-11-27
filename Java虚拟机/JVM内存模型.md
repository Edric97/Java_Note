# JVM内存模型

首先看一下本机的JVM是什么：

```markdown
-bash-3.2$ java -version
java version "1.8.0_251"
Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
```

很好，我的是HotSpot虚拟机。



接下来，为了方便自顶向下的理解内存模型，我们尝试从线程的角度去认识。



首先我们明确，运行当前main()方法下的类是一个进程，里面有分成了若干个线程，之前学习JavaSE的时候，写的程序多半只有一个main()线程，即使在main()方法中引用了别的类的方法。比如在下面测试StackOverflowError的时候：

```java
public class TestStackOverflowError {
	public static void main(String[] args) {

		System.out.println("当前为主线程： " + Thread.currentThread().getName());

		Backup backup = new Backup();
		backup.dfs();
	}
}

class Backup {
	public void dfs() {
		System.out.println("当前线程为： " + Thread.currentThread().getName());
		dfs();
	}
}

```

上面的不管是主线程还是下面dfs()中的线程，线程名都是main。



但是要明确的一点是，一个线程会包含若干的方法，比如下面的测试代码：

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

其结果为：

```markdown
-bash-3.2$ java TestThread
当前主线程名为： main
在住线程中被调用，线程名为： main
启动一个字线程，线程名为： 我是子线程01
在run方法里面被调用，线程名为： 我是子线程01
在runSub方法里面被调用，线程名为： 我是子线程01
```



因此从线程的角度去理解内存模型是没有问题的，因为你运行程序的时候，比方说main()方法中用到了别的线程，那么这个“别的线程”同主线程一样，会在内存中开辟一个三个空间（程序计数器、Java虚拟机栈、本地方法栈）去存放这个线程应该有的控制运行流程的功能的程序计数器、以及线程用到的方法存在在Java虚拟机栈、线程用到的底层方法存放在本地方法栈。



现在看一下，我的HotSpot虚拟机的Java虚拟机栈最多能放多少个栈帧（一个方法对应一个栈帧）：

```java
class TestStackOverflowErrorContent {

	static int count;

	public static void main(String[] args) {
		dfs();
		System.out.println(count);
	}

	public static void dfs() {
		try {
			++count;
			dfs();
		} catch (StackOverflowError e) {
			return;
		}
	}
}

```

其结果是：

```markdown
-bash-3.2$ java TestStackOverflowErrorContent
21691
```



虽然线程很重要，但是不同的线程总归是会共享同一个类的。仍然以下面的代码为例：

```java
class TestThread extends Thread {
  int ownTogether = 1;

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
		System.out.println("在主线程中被调用，线程名为： " + Thread.currentThread().getName());
	}
}


```

无论是主线程还是用到的别的线程，都会共享TestThread类中的`ownTogether`全局变量。这些线程共有的东西，内存中会开辟两个区域，一个是用来存放对象实例的堆空间，一个是方法区,（元空间）也叫做“非堆”（？？？？？？存放了啥？？？？？？）——“非堆”的叫法，也凸显出了堆空间的重要性！



符号引用就是加载class文件进内存的时候，（所谓的加载机制，就是把.class文件放进内存，然后解析生成对应的class对象的一个过程）JVM不知道引用的对象在内存的具体的地址，所以暂时先用一个唯一的符号表示这个对象，等到后面解析的时候再把符号引用转换为直接引用（此时JVM找到了引用对象的地址，也是因为Java代码只会到 虚拟机加载Class文件的时候 进行动态连接）。因此常量池在Class文件中，



堆空间大小 = 年轻代空间大小 + 老年代空间大小

-XX:+PrintGC
-XX:+PrintGCTimeStamps
-XX:+TraceClassLoading

-Xms设置堆空间初始大小

-Xmx设置堆空间最大大小

-Xmn设置年轻代大小

-XX:MetaspaceSize=10m 设置元空间发生GC的初始阈值为10m

-XX:MaxMetaspaceSize=20m 设置元空间的最大空间的大小

-Xss 设置单个线程栈大小





魔数、主次版本号、常量池计数器、常量池（字面量、符号引用）



一个16进制数占4位，因此两个16进制占1个字节



常量池计数器因为是无符号整数，且u2（代表占2个字节），因此最多只能容纳2 ^ 16（因为无符号整数u2有16位，即[0, 2 ^ 16 - 1]，包括0一共2 ^ 16个数据）个常量