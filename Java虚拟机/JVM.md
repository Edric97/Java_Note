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


虽然普通的 static 变量需要在类的初始化时被赋值，但是 static final 变量在前端编译成Class文件的时候，就已经被赋值在Class文件的字面量中。


字面量包括文本字符、声明为final的常量值、基础数据类型的值，字符引用包括类或接口的全限定名、字段的名称或描述符、方法的名称或描述符。

类的全限定名就是：包名.类名。（但是不包括访问修饰符，诸如public protected等等，这些在后面的访问标志那里保存的，可以通过命令javap -public 只显示public类以及成员）

例如：程序

```java
class TestThreadSecurity {
	public static void main(String[] args) {
		Thread t1 = new Thread01();
		Thread t2 = new Thread01();
		t1.start();
		t2.start();
	}
}

class Thread01 extends Thread {
	public int test = 1;
	static int count = 10;
	@Override
	public void run() {
		if (count < 0) {
			return;
		}
		System.out.println("当前线程为： " + Thread.currentThread().getName());
		while (count >= 0) {
			System.out.println("当前线程 " + Thread.currentThread().getName() + " count值为： " + count--);	
		}
	}
}

```

前端编译成.class文件后，使用命令`javap -public TestThreadSecurity`显示如下：

```markdown
Compiled from "TestThreadSecurity.java"
class TestThreadSecurity {
  public static void main(java.lang.String[]);
}
```

Class文件中，在常量池后面存储的是访问标志（就是标识类或接口的访问信息，诸如是否public，是否为接口，是否为注解，是否为枚举，是否final等）。访问标志后面则是类索引、父类索引，值得注意的是，虽然类、父类索引是用来确定类或者父类的全限定名，但是不是说这里u2的数据存储的就是全限定名，而只是一个索引，该索引指向了常量池中的CONSTANT_Class_info，而CONSTANT_Class_info也不是直接存的全限定名，而是又指向了常量池中的CONSTANT_Utf8_info，这里才是真的存储了全限定名。在类索引、父类索引的后面，存储的则是接口索引计数器、接口索引集合，计数器存储的是接口索引集合中接口的数量，接口索引集合则按序每个u2大小存储接口的信息。




在接口索引的后面，则是字段表、方法表、属性表。但是这三个表，各自在前面跟了一个不可分割的计数器，分别为字段表计数器、方法表计数器、属性表计数器，分别用来记录数量。比如说一个类：
```java
public class JVM {
	private String test;

	public String getName() {
		return test;
	}

	public void setName() {
		this.test = "zhangsan";
	}
}
```
对这个类来说，字段表会存储的信息为`private String test`。
方法表存储的信息为：`public String getName` `public void setName`。
属性表存储的信息为: `name`。
很明显，字段和属性是不一样的，因为属性是JavaBean的一个规范，不是类中提出的概念。类的全局变量称之为字段，属性是在JavaBean（满足字段 + get/set方法等的规范的类）中get/set方法后面跟着的那个东西，只是很多时候，为了容易开发，所以把属性名和字段名写的一样。但是比如上面这个程序中，就只有test字段名，name属性名。



多说几句：
为了满足JavaBean规范，所以我们定义类的字段名，一般头两个字母统一小写，或者统一大写。不然在生成get/set方法时，比如定义字段`aGe`，则生成的get方法名为`getaGe`，这样Spring反射得不到`aGe`属性。




回顾一下，Class文件中存储了哪些东西：
```markdown
魔数 ——》主次版本号 ——》常量池计数器 ——》常量池 ——》类索引 ——》父类索引 ——》接口索引计数器 ——》 接口索引集合 ——》字段表 ——》方法表 ———》属性表
```



常量池存储的全是常量，

```java
class JavaBean{
    private int value = 1;
    public String s = "abc";// “abc”对应常量池中字面量， `public String s`对应常量池中的字符引用
    public final static int f = 0x101;

    public void setValue(int v){
        final int temp = 3;
        this.value = temp + v;
    }

    public int getValue(){
        return value;
    }
}
```



`"abc"（文本字符串）` `0x101(257)（被final修饰的成员变量，包括静态变量、实例变量、局部变量）`均存储在常量池的字面量（也就是具体数据的值），而前面的`String` `final static`则保存在常量池的符号引用，修饰符在后面的字段表中才会被连上。但是`private int value = 1`，这一项，常量池中只有`I描述int`以及`value`，数值以及修饰`private`（因为修饰符只有存在或者不存在两种状态，很适合放在字段表中用标识位来表示）则存放在后面的字段表中。（对final静态变量而言，字段表中存的是引用，指向了常量池中的数值）。



解释一下符号引用中的描述符，是这样的：

`int`的描述符是`Int`

`double`的描述符是`Double`

基本数据类型除了`long -> J ` `boolean -> Z`，其余全是首字母大写。

`String`的描述符是`Ljava/lang/String`，是全限定名前加`L`。

数组的描述符，诸如`int[]`的描述符是`[I`。





常量池到底存了啥？

```java
package a.b;
public class Test {
  final int t1 = 1;
  int t2 = 2;
  static int t3 = 3;
  final static int t4 = 4;
  String s = "abc";
}
```

很明显，常量池中的存的是`int t1（前面的都是字符引用） 1（字面量）` `int t2` `int t3` `int t4 4` `s:Ljava/lang/String s（前面的都是字符引用） "abc"（字面量）`，以及`class ` `a/b/Test（全限定名）`  （像Object以及构造方法啥的就不写粗来了）。这些东西的共同点就是不会再变化了。叫做常量池，因为常量不会再变化了。作为类的`public`则单独放在访问标志中，即设置标志值`0x0001`。



常量池就是个池子，后面需要的话，就会从池子里面取出来。

可以这么说，除了修饰符，以及那些可能会变化的量（比如实例变量的值），其他元素，全部在常量池中。所以说常量池是Class文件中，占用空间最大的项目之一。其他的访问标志（这个不引用常量池）、字段表、方法表、属性表（均引用了常量池里的东西，因为这三个表里特别的信息基本都是修饰符等有限状态的，像变量名，返回值类型，类名等不是有限类型的，就存放在常量池里，然后这三个表去引用）。
