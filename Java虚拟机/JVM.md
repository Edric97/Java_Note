# JVM



学习时，以`.class`文件作为抓手！



因为不管JVM怎么运行，信息全部存储在`.class`文件中，都是围绕着`.class`文件需要时怎么被装进内存（类的加载机制），内存中又是怎么划分的（运行时数据区），以及怎么执行具体的方法（执行引擎）。



而`.class`文件中，我觉得最重要的就是常量池。



因为大部分别的信息，都是引用这个常量池的东西。





类的加载机制的过程是`加载、链接、初始化`，结果是`.class`文件被装进内存中的方法区，此时`.class`文件中作为字符串形式出现的字符引用要全部转换为直接引用，这个过程就是链接里面的解析过程（细化一下就是常量池、域信息_对应字段表、方法信息_对应方法表、class类元信息_对应访问标志），并且在堆区生成了对应的Class对象（类似于一个接口，指向了方法区的类的结构）。创建对象实例的时候，就是使用的是那个Class对象（不会直接使用方法区中的Class结构，通过Class对象找到方法区中的Class结构，这也就是反射）。执行类的方法使用执行引擎来运算，产生的数据则会放在虚拟机栈，用到了别的类或者方法，要从常量池中通过引用拿数据（基本单位是栈帧，栈帧可以细化为局部变量表、操作数栈_栈中栈哈哈哈哈、动态链接、方法返回值）这个内存空间中。但是用完的类或者方法，不能一直占用内存吧。所以需要垃圾回收机制，去动态的管理清除内存。



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



```markdown
-X 是JVM的运行参数
```





堆空间大小 = 年轻代空间大小 + 老年代空间大小

-XX:+PrintGC
-XX:+PrintGCTimeStamps
-XX:+TraceClassLoading

-Xms设置堆空间（这里只算上年轻代和老年代）初始大小 (ms 是memory start)

-Xmx设置堆空间（这里只算上年轻代和老年代）最大大小

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


现在我们已经有了.class文件的，并且对Class文件包含的信息以及存储的结构已经有了更清晰的认识。现在，我们思考，如何把Class文件从外存加载到内存呢？（从内存到CPU运行，是执行引擎的事情，不在现在的讨论范围之内）。


首先，要把类的十六进制转换为二进制（注意，类名一定是全限定名，参见下面一段的例子），然后把该class的静态结构转换为运行时数据区的动态结构，再然后在堆空间中生成Class对象（往后，new出来的实例对象全部从该Class对象获取信息并创建）。生成Class对象是类的加载的加载阶段的最终目标！

但是如何在内存（堆空间）中创建出这个Class对象，这个就需要类加载器来操作了。
比方说我们自定义了一个String类，但是在这个自定义的String类里面，想用java.lang.String类，例如`String s = "abc"`，此时就会报错，因为按照双亲委派模型，系统类加载器往上去扩展类加载器去找，然后扩展类加载器去找启动类加载器，启动类加载器发现这个自定义的String类不在它的管辖范围，于是往下派送给扩展类加载器，但是扩展类加载器发现也不在它的管辖范围，于是只能有系统类加载器来加载了，因此`String s = "abc"`会被JVM认为是自定义的String类。因此要是想用java.lang.String，一定要写上全限定名，才能被启动类加载器加载到内存中！即`java.lang.String s = "abc"`。
再从Class 文件结构开始仔细剖析一下`java.lang.String s = "abc"`这句话，首先`java.lang.String`以及`s`以及`"abc"`会被放在常量池中，然后在类的加载的加载阶段，`java.lang.String` `s` `"abc"`三个字符串在Class文件中以16进制存在，会被转换为2进制，然后放在运行时数据区，并且通过双亲委派机制生成`java.lang.String`的Class对象在堆中。在后面初始化的时候，会从`java.lang.String`的Class对象去创建对象，并从自定义的String类的Class对象的常量池中去取"abc"这个值。



没有初始化的时候，堆空间中是没有实例化的对象的，都是被加载的Class对象。





例如：java.lang包下的所有类全部在启动类加载器的管辖范围内





类加载器的作用就是：通过类的全限定名来获取.class文件流的工具



执行的永远是方法，即CPU处理的是方法。而类只要存储在内存即可，不和CPU直接打交道。

类的加载过程，只是加载了类本身的信息。创建对象，是另一个东西了。



类的加载过程一共有三个阶段：

```markdown
1、加载阶段：（个人觉得是最主要的步骤，毕竟加载进内存是在这个阶段完成的，后面的都是复制啥的）
		将类的字节码文件加载到机器内存中（类的加载器做的事情），（处理二进制信息前会验证）并在内存中构建出Java类的原型——类模版对象，即Class对象（所谓类模版对象，就是Java类在JVM内存中的一个快照，JVM将从字节码文件中解析出的常量池、类字段、类方法等信息存储到类模版中，这样JVM在运行期间可以通过类模版而获得Java类中的任意信息，能够对Java类的成员变量进行遍历，也能进行Java方法的调用）
2、链接阶段：
		验证：验证这个.class文件是不是合格的，包括四个方面：文件格式验证、元数据校验、字节码验证、符号引用验证
		准备：将一般的static变量赋0值，final static是常量，因此显示赋原本定义的值
		解析：将符号引用转换为直接引用（因为此时类、接口、方法等在内存中的位置已经确定了）
3、初始化阶段
		真正的初始化static变量（就是执行<clinit>方法的过程，这个方法不是程序员定义的，是编译器自动生成的，用来赋值static变量以及static代码块）
```



继续解释加载阶段：

类将.class文件加载到元空间后，会在堆中创建一个java.lang.Class对象，用来封装类位于方法区内的数据结构，该Class对象是在加载类的过程中创建的，每个类都对应有一个Class类型的对象。

即元空间中，是.class文件的数据结构，堆区是.class文件的Class对象。

java.lang.Class实例是访问类型元数据的 `接口`，也是实现反射的关键数据、入口。通过Class类提供的接口，可以获得目标类所关系的.class文件中具体的数据结构：方法、字段等信息。



```markdown
值得注意的点：
在链接的验证阶段，又细分成了四种验证（文件格式验证、元数据验证、字节码验证、符号引用验证）。其中，文件格式验证和加载阶段一起执行。验证通过后，类加载器才会将已经被找到的类的二进制数据信息加载到方法区中。而文件格式验证之外的验证操作将会在方法区中进行。
```



现在终于算是明白了！先是类的加载器去目录或者某种渠道找到.class文件，然后验证是否符合.class的格式要求。要是符合的话，类的加载器就把这个类加载到方法区，并且在堆区生成一个Class对象作为方法区这个类的接口（因为不会允许直接访问方法区，所以加了这一层，访问类的具体信息，都是通过Class对象来访问方法区中类的信息）。再然后就是在方法区中，做其他的验证。再后面就是准备、解析和初始化。





运行时数据区：Java运行的基石是类，因为方法区很重要，而一开始.class文件在外存，因此要用类的加载机制加载到方法区中，并在堆空间中生成对应的Class对象作为别人访问这个类的`接口`。运行这个类，你需要new一个对象，因此我们用被加载进内存的类的对应的Class对象来创建新的实例。比方说，我们的main线程用到了某个方法，于是我们将该方法的所有信息压入虚拟机栈，通过局部变量表、操作数栈等栈帧的组成部分以及执行引擎、PC寄存器合力将结果运算出来。最终返回需要的结果，然后将上面所有用到的数据全部从内存中清除。



Java里的线程和操作系统的线程是有一个一一对应的关系的



PC寄存器就是用来存储下一条指令的地址的

模拟了硬件上的PC寄存器





栈也没有垃圾回收机制



为什么要使用PC寄存器来记录当前线程的执行地址？

```markdown
因为CPU不断的切换各个线程，切换回来的时候，需要知道从哪里开始。JVM的字节码解释器需要通过改变PC寄存器的值来明确下一条应该执行什么样的字节码指令。
```





栈管运行（和方法紧密相关），堆管存储



一个栈对应一个线程，一个栈帧（栈中的数据都是以栈帧的格式存在）对应一个方法。



栈什么时候会出现OutOfMemoryError呢？

```markdown
虚拟机栈可以动态扩展，并且在尝试扩展时无法申请到足够的内存，或者创建新线程时没有足够的内存去创建虚拟机栈，则会出现OOM异常。
```



StackOverflowError呢？可以设置栈的深度。-Xss



既然知道栈是由栈帧构成的，那么栈帧是由什么构成的呢？

```markdown
局部变量表（存储的是方法的参数以及局部变量，局部变量又分为基本数据类型、引用类型、返回值）（由局部变量槽以数组形式存在）、操作数栈、为了实现动态链接的 引用 （指向了Class文件的常量池）、方法返回地址、附加信息
```

注意：this也要存放在局部变量槽中



byte, short, char, boolean 都以int型来存储 



操作数栈：栈帧里面的栈^_^，就是在方法执行的时候，根据字节码指令，往操作数栈中写入数据或提取数据，运算则交给执行引擎



给一段代码：

```java
byte i = 15;
int j = 8;
int k = i + j;
```

javap后

```markdown
0:bipush 15 // 此时在操作数栈中压入15，但是局部变量表什么都没有，0表示的是PC寄存器指向的指令
2:istore_1 // 此时操作数栈什么都没有，但是局部变量表存储了15
3:bipush 8
5:istore_2
6:iload_1 // 从局部变量表中取出第一个数据，即15，到操作数栈
7:iload_2 // 从局部变量表中取出第二个数据，即8，到操作数栈
8:iadd // 把两个数据出操作数栈，是个字节码指令，需要执行引擎翻译iadd为机器指令并做运算，结果压入栈
9:istore_3 // 操作数栈取出，此时什么也没有。局部变量表中保存第三个数据，即结果23.（但是注意的是，局部变量表的最大长度是4，因为索引为0表示的是this）
10:return
```



（左值，等号左边的，内存中有地址的值，比如）

```java
var a;
a = 3;
```

a就是左值，但是

```java
3 = a;
```

就不行，因为3不会在内存中占据空间。



++i是个左值，++i = 1可以，i++则不是，i++ = 1不行，它要先找到个值赋上去然后再加1。++i则直接运算，不用再找个值。





栈顶缓存技术：就是操作数栈栈顶的元素缓存到物理CPU里，这样可以避免频繁的IO操作，提高了运算速度。



栈帧中的方法返回地址存放的是调用该方法的PC寄存器的值。（比如，递归回溯的时候）





一个进程对应一个JVM的实例



堆区在JVM启动时被创建，且堆的大小在启动时被确定（可以用-Xms -Xmx 设置堆区的大小），那么JVM什么时候被创建呢？——被启动类加载器加载的时候就创建了。



堆可以物理上不连续，但是逻辑上应该是连续的。



因为用户线程和垃圾回收线程（守护线程）是不能一起执行的，所以垃圾回收的时候，用户线程会暂停执行。为了不让用户线程总是因为垃圾回收而暂时影响效率，因此垃圾回收不是说对象没有引用的时候就立马被回收，而是在垃圾手机（GC）的时候才会被回收。



IDEA装插件jClassLib，用来反编译类的代码。



细分堆空间：（因为堆空间分代了，所以垃圾回收也会基于分代理论）

JDK8，堆空间内存逻辑上分为三部分，分别是`年轻代、老年代、元空间`

JDK7以及之前，堆空间的内存逻辑上分为`年轻代、老年代、永久区`

但是其中元空间、永久区，一般不在管辖范围之内。

（有点类似于中国的台湾）

而年轻代（占用的最大内存通过`-Xmn`设置最大比例）又可以细分为`Eden（伊甸园）空间，Survivor0和Survivor1空间（有时又被称为from区、to区）`，并且在HotSpot中，Eden区和另外两个Survivor区的比例默认是8:1:1，当然这个比例同样可以调整，比如通过命令`-XX:SurvivorRatio=10`设置比例为10:1:1。



几乎所有的对象都是在Eden区被new出来的。



GC频繁在Eden区收集，很少在老年代收集，几乎不在永久区或者元空间收集。





想让程序跑的时间更长一点，可以用`Thread.sleep(100000)`。





IDEA的`edit configurations`可以设置当前类的JDK版本





`Throwable` 的两个子类  `Error` `Exception`



存储在JVM中的对象可以被划分成两类：

```markdown
一类是生命周期较短的瞬时对象，这类对象的创建和销亡都非常迅速
一类生命周期非常长，某些极端的情况下，还能与JVM的生命周期保持一致（比方说用来连接的对象）
```





年轻代和老年代在堆空间中各自所占的比例是可以通过参数调整的，例如使用`-XX:NewRatio=2`表示年轻代占比1，老年代占比2，即年轻代占堆空间的1/3。但是实际开发中，我们一般不会去调整这个参数。但是比方说，你事先知道了程序中，有很多对象的生命周期都很长，这个时候就可以将老年代的比例设置的更大。





为新对象分配内存？

```markdown
要考虑的问题有很多，比方说如何分配内存，在哪里分配，如何让GC完之后不产生内存碎片。
```

先扔在Eden区，当Eden区满的时候，触发GC机制（但是Survivor区满的时候，不会触发GC机制）（特指youngGC或者称作minor GC），此时Eden区是空的，然后存活的扔到from区，并给这些对象的年龄计数器加1，新对象扔到Eden区，此时Eden区不是空的。再次GC的时候，要是from区满的话，就扔到to区，否则仍然扔到from区，并给这些对象的年龄计数器加1，然后from区存活的对象扔到to区，这些对象的年龄计数器加1。判断对象的年龄计数器达到一定的标准的时候（默认是15次，即当年龄计数器为16的时候就在老年代里面了，但是可以通过参数`-XX:MaxTenuringThreshold=某个数值`来设定），就将年轻代的对象扔进老年代。

但是存在从Survivor区（即触发YGC的时候，Survivor区已经满的时候，会直接扔进老年代，但是要是这个对象太大了，老年代也容纳不下，就会触发FGC，此时判断老年代是否可以放得下该对象，放得下的话就扔在老年代，否则报OOM）或者Eden区直接到老年代（YGC后，Eden区仍然放不下，尝试扔到Survivor区，仍然放不下，直接扔到老年代）的情况。



同步：发出请求之后，等待返回后，再发送下一个请求（流水线的感觉）

异步：发出请求之后，不等待返回，随时可以发送下一个请求

```markdown
同步和异步最大的区别就在于。一个需要等待，一个不需要等待。
比如广播，就是一个异步例子。发起者不关心接收者的状态。不需要等待接收者的返回信息
电话，就是一个同步例子。发起者需要等待接收者，接通电话后，通信才开始。需要等待接收者的返回信息
```





现在我们来看一下，当我们说到对象的时候，并且给对象分配内存的时候，内存里的对象具体是些什么东西，即对象是由底层的什么构成的？

```markdown
1、对象头：包含两类信息
	一类信息是：存储对象运行时的特征数据（就是说，这段逻辑上连续的空间的特征数据。物理上不连续的空间，使用指针连接。）
		HashCode、GC用的年龄计数器、锁状态标志、线程持有的锁等等
	还有一类信息是：类型指针（JVM通过这个指针来确定该对象是哪个类的实例，指向了方法区）
2、实例数据：是对象真正存储的有效信息（就是程序代码里面定义的各种类型的字段内容）
3、对齐填充：（于HotSpot虚拟机的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说就是 任何对象的大小都必须是8字节的整数倍。对象头部分已经被精心设计成正好是8字节的倍数（1倍或者2倍），因此，如果对象实例数据部分没有对齐的话，就需要通过对齐填充来补全）
```



贴一篇文章：https://segmentfault.com/a/1190000038738567



使用对象的话，使用虚拟机栈里面的局部变量表里的引用即可。但是这个引用有两种情况，一种是直接找到堆空间中对象实例的数据（该对象实例里面保存的类型指针则会指向方法区中的对象类型数据）；第二种则是存储的是对象的句柄地址，这个句柄（相当于一个表的形式）包含了对象实例数据与类型数据各自具体的地址信息。



分配内存的话，是和具体虚拟机的具体垃圾回收机制有关系，要是垃圾回收机制收集完内存后，顺带整理一下内存（这个是压缩整理算法）（例如Serial、ParNew），让内存一边是空的，一边是被占用的，则使用一个指针，指向空的和被占用的中间的地址，下次直接移动这个指针即可（这个叫做指针碰撞方式）；要是垃圾回收完不管内存空间的整理（这个是基于sweep算法的垃圾收集器，比如CMS），就要维护一个表，记录哪些内存块是可用的，分配的时候查表，从表中找到一块足够大的空间划分给对象实例，并更新表上的记录（这个叫做空闲列表方式）。



![手画方法运行时示意图](/Users/caoxiaodong/Desktop/手画方法运行时示意图.jpeg)



