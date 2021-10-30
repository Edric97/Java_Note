# AOP——面向切面编程

AOP的作用是解耦合，即对业务逻辑的各个部分进行隔离，比方说：

```markdown
在写书城项目的时候，登录功能完成后，还想在原方法里加上权限验证功能。
如果采用最传统的方式进行的话，我们还要找到原方法，然后源代码上修改，这样不好维护，万一新增的代码写歪了，找BUG就很难了。
但是利用AOP的话，则可以把权限验证功能单独拉出来写，然后直接配置到源代码上。这样即使写歪了，也容易排查，即使不用这个功能了，也不用修改源代码。
这样就做到了耦合度的降低。
```

## 一、AOP底层原理

底层使用了动态代理。

```markdown
有接口 —— JDK动态代理
无接口 —— CGLIB动态代理
```

### 1、有接口 ——传统

传统：

```java
interface UserDao {
	void login();
}

class UserDaoImpl implements UserDao {
	public void login() {
    //代码逻辑
	}
}
```



创建UserDao接口实现类的代理对象

注：代理对象不是new出来的

### 2、无接口 —— 传统

传统：

```markdown
class User {
	void add();
}

class Person extends User {
	public void add() {
		super.add();
		//增强逻辑代码
	}
}
```



创建当前子类的代理对象

注：代理对象不是new出来的

### 3、JDK动态代理实现

首先明确是由 `java.lang` 包下的 `Proxy` 类里的 `newProxyInstance(ClassLoader loader, 类<?>[] interfaces, InvocationHandler h)` 方法实现的。

其中` 类<?>[] interfaces` 就是增强方法所在的类，该类实现的接口，支持多个接口； 而`InvocationHandler`也是一个接口，创建了代理对象，并在这里写增强方法的代码。

```java
public interface UserDao {
	int add(int a, int b);
  String update(String id);
}
```



```java
public class UserDaoImpl implements UserDao {
	@Override
  ...
    
  @Override
  ...
}
```



```java
//创建代理对象代码
class UserDaoProxy implements InvocationHandler {
	//把被代理的对象传过来
  //有参构造传递
  private Object obj;
  public UserDaoProxy(Object obj) {
    this.obj = obj;
  }
  
  //增强的逻辑代码部分
  public Object invoke(...) ... {
		//方法前
    ...
      
    //被增强的方法的原方法
    Object res = method.invoke(obj, args);
    
    //方法后
    ...
      
    return res;
  }
}
```



```java
public class JDKProxy {
  Class[] interfaces = {UserDao.class};
  UserDaoImp userDao = new UserDaoImpl();
  
  Proxy.newProxyInstance(JDKProxy.calss.getClassLoader, interfaces, new UserDaoImpl(userDao));
}
```



## 二、一些术语

```markdown
连接点：可以被增强的方法
切入点：实际上被增强的方法
通知（增强）：实际上增强的逻辑部分
	有如下几种类型：
  	前置通知 @Before
    后置通知 @AfterReturning
    环绕通知 @Around
    异常通知 @AfterThrowing
    最终通知 @After
切面：是个动作，指的是把通知应用到切入点的过程
```



## 三、AOP准备

### 1、明确

首先明确：

```markdown
Spring框架一般是基于 Aspects 实现AOP操作的
```

这里的`Aspects`也是个框架，是独立的，不是Spring里面的。

同样的AOP的实现方式也有两种：基于xml配置文件，基于注解方式。

下面只说最常见的基于注解的实现方式。

### 2、准备

首先引进依赖，即jar包。

在xml配置文件中写：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="spring5_aop"></context:component-scan>
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

### 3、一个介绍

简单说一下“切入点表达式”（知道对哪个类中的哪个方法进行增强），其语法结构为：

```markdown
execution([权限修饰符][返回类型][类全路径][方法名称]([参数列表]))
```

## 四、实例

准备好了，直接上手写一个小实例。

首先还是看一下层次结构：

```markdown
src 
	spring5_aop包
		User类
		UserProxy类
		Test类
bean4.xml
```

其中`bean4.xml`就是准备工作中的配置文件，在这里还是重新放上：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd
                            http://www.springframework.org/schema/context http://www.springframework.org/schema/context/spring-context.xsd
                            http://www.springframework.org/schema/aop http://www.springframework.org/schema/aop/spring-aop.xsd">

    <context:component-scan base-package="spring5_aop"></context:component-scan>
    <aop:aspectj-autoproxy></aop:aspectj-autoproxy>
</beans>
```

然后是`User`类：

```java
package spring5_aop;

import org.springframework.stereotype.Component;

@Component
public class User {
    public void add() {
        System.out.println("add()方法执行了");
    }
}

```

再然后是`UserProxy`类：

```java
package spring5_aop;

import org.aspectj.lang.annotation.Aspect;
import org.aspectj.lang.annotation.Before;
import org.springframework.stereotype.Component;

@Component
@Aspect
public class UserProxy {
    @Before(value = "execution(* spring5_aop.User.add(..))")
    public void before() {
        System.out.println("增强逻辑部分执行了");
    }
}

```

最后是测试类：

```java
package spring5_aop;

import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;

public class Test {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean4.xml");
        User user = context.getBean("user", User.class);
        user.add();
    }
}

```

最后测试类的输出如下：

```markdown
增强逻辑部分执行了
add()方法执行了

Process finished with exit code 0
```

