# IOC容器——Bean管理（基于xml文件）

## 一、概述

### 1、什么是Bean管理

指的是两个操作：

```markdown
1、Spring创建对象<bean>
2、Spring注入对象<property>
```

——注释：

注入对象，指的就是给类的实例域赋值。

以上这两个操作交给Spring框架完成即可，不用自己手动 new() 对象等等操作。

本次笔记讲的全部都是基于xml配置文件的，实现`Spring创建对象`和`Spring注入对象`的这两个操作。

### 2、一点自己的想法

```markdown
IOC就是将Bean对象和上层隔离开了，这样上层用到了你这个Bean对象，一旦Bean对象发生了改变，也不需要在上层找到你这个Bean对象的相关信息，然后在上层修改。
相当于说，这个xml文件就是一个“工厂”（理解这一点很重要）：
	这个工厂生产的就是xml文件里<bean>标签里的对象。而用户/上层用到这个Bean对象的话，先xml解析得到这个Bean对象的字节码文件名，然后通过反射的newInstance()方法生产出这个类的实例。
```

### 3、Bean管理操作的两种方式

```markdown
1、基于xml配置文件实现
2、基于注解方式实现
```

——注释：

暂时先解释一下 基于xml配置文件实现：

```xml
<bean id="" class=""> —— 创建对象（默认是通过无参构造器构造的）
  <constructor-arg name="" value=""></constructor-arg> —— 加上这一行可以使用有参构造器，并给注入参数给对象（不过不常用）
  <property name="" value=""></property> —— 这个是通过原类的set()方法注入对象的，且这个是一般类型的
  <property name=""> —— 这个注入的是List列表
    <list>
      <value></value>
    </list>
  </property>
  <property name=""> —— 这个注入的是数组类型
    <array>
      <value></value>
    </array>
  </property>
  <property> —— 这个注入的是Map类型
    <map>
      <entry key="" value=""></entry>
    </map>
  </property>
  <property> —— 这个注入的是Set类型
    <set>
      <value></value>
    </set>
  </property>
</bean>
```

## 二、Spring创建对象

就是在xml配置文件里面，用<bean>标签创建。

其中<bean>标签里面，最重要的两个属性分别是`id`和`class`。

`id`属性指的是，这个类的别名，也就是唯一标识，在上层引用该xml文件创建Bean对象的时候，就是用的这个别名，比如`User user = context.getBean("user", User.class);`

`class`属性指的是，这个类在src下的路径，是类的全路径，包括了包名。

举一个例子：

```xml
<bean id="user" class="spring5.User"></bean>
```

## 三、Spring注入对象

首先明确一下，注入方式有两种，一种是通过`有参构造器`往里面填值，一种是通过`set()`方法往里面设置值。

### 1、有参构造器注入

#### 1.1 原Bean类里面

照常写有参构造器

#### 1.2 xml配置文件里面

根据原Bean类里面的有参构造器方法里的参数，填入值。

即，在<bean>标签的内容里用<constructor-arg name="" value="">标签来填值。

例如：

```xml
<bean id="order" class="spring5.Order">
  <constructor-arg name="oname" value="Jack"></constructor-arg>
  <constructor-arg name="address" value="Beijing"></constructor-arg>
</bean>
```

——注释：

如果<bean>标签里面没有<constructor-arg>标签，默认是使用的无参构造器，一旦填上了<constructor-arg>标签，并且参数个数和名称内容准确对上原Bean类里面的有参构造方法，就会使用原Bean类的那个有参构造方法。

### 2、set()方法注入

#### 2.1 原Bean类里面

正常写针对实例域的`set()`方法。

#### 2.2 xml配置文件里面

在<bean>标签的内容里面，用<property>标签设置值。

例如：

```xml
<bean id="book" class="spring5.Book">
  <property name="bname" value="HarryPotter"></property>
</bean>
```

下面写两种特殊的实例域：

##### 2.2.1 null值

```xml
<property name="address">
  <null/>
</property>
```

##### 2.2.2 含有特殊字符

除了可以用转义字符，还可以用`CDATA`

```xml
<property name="address">
  <value>
    <![CDATA[<<Hello>>]]>
  </value>
</property>
```

### 3、P名称空间——简化注入

如果采用的是`set()`方法注入，那么要在xml文件中写一堆<property>标签，但是这里是可以简化的。

即可以将属性直接写在<bean>标签中，这就被称作 P名称空间。

step1：添加P名称空间在配置文件中的约束

```markdown
xmlns:p="http://www.springframework.org/schema/p"
```

step2：在xml配置文件中写

```xml
<bean id="book" class="spring5.Book" p:bname="HarryPotter" p:author="JKR"></bean>
```

### 4、外部注入、内部注入和级联赋值

上面注入的属性都是很常见的数据，比如基本数据类型、String等，要是注入的属性是自定义类又该怎么注入呢？

这里就有两种方式，下面直接写例子，来的更直观：

#### 4.1 外部注入

```xml
<bean id="userService" class="spring5.UserService">
  <property name="userDao" ref="userDaoImpl"></property>
</bean>
<bean id="userDaoImpl" class="spring5.UserDaoImpl"></bean>
```

#### 4.2 内部注入与级联赋值

```xml
<bean id="emp" class="spring5.Emp">
  <property name="ename" value="Jack"></property>
  <property name="gender" value="male"></property>
  <property name="dept">
    <bean id="dept" class="spring5.Dept">
      <property name="dname" value="Tech"></property>
    </bean>
  </property>
</bean>
```

### 5、集合的注入

问题：怎么注入数组？List集合？Map集合？

#### 5.1 集合注入的例子

```xml
<bean id="" class=""> —— 创建对象（默认是通过无参构造器构造的）
  <property name=""> —— 这个注入的是List列表
    <list>
      <value></value>
    </list>
  </property>
  <property name=""> —— 这个注入的是数组类型
    <array>
      <value></value>
    </array>
  </property>
  <property> —— 这个注入的是Map类型
    <map>
      <entry key="" value=""></entry>
    </map>
  </property>
  <property> —— 这个注入的是Set类型
    <set>
      <value></value>
    </set>
  </property>
</bean>
```

#### 5.2 集合注入的细节

有两个问题，即`如何在集合里面设置对象的类型值`以及`如何将集合注入的某个部分提取出来`

##### 5.2.1 如何在集合里面设置对象的类型值

```xml
<property name="courseList">
  <list>
    <ref bean="course1"></ref>
    <ref bean="course2"></ref>
  </list>
</property>

<bean id="course1" class="spring5.Course">
  <property name="ename" value="Math"></property>
</bean>
<bean id="course2" class="spring5.Course">
  <property name="ename" value="English"></property>
</bean>
```

##### 5.2.2 如何将集合注入的某个部分提取出来

这个问题的实际意义在于，解耦合，不然每次创建实例，都要重写一段<property>的<value>标签

step1：在xml配置文件约束中，引入名称空间util

```xml
xmlns:util="http://www.springframework.org/schema/util"

xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
http://www.springframework.org/schema/util
http://www.springframework.org/schema/util/spring-util.xsd
"
```

step2:使用util标签完成list集合注入提取

首先提取公共部分：

```xml
<util:list id="bookList">
  <value></value>
</util:list>
```

然后注入：

```xml
<bean id="book" class="spring5.Book">
  <property name="list" ref="bookList"></property>
</bean>
```

