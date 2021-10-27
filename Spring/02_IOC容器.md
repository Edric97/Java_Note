# IOC容器

从三点论述IOC容器：

```markdown
1、IOC底层原理
2、IOC接口
3、IOC下的Bean管理
	基于xml
	基于注解
```

## 一、概念和原理

### 1、什么是IOC

```markdown
控制反转，将对象创建和对象之间的调用过程，交给Spring进行管理
```

### 2、使用IOC的目的：

```markdown
降低耦合度
```

## 二、底层原理

### 1、底层原理是哪些

```markdown
xml解析、工厂模式、反射
```

其中工厂模式，是设计模式中的一种

### 2、一个实例

#### a.最开始的情况

Dao层：

```java
class UserDao {
  add() {
    ...
  }
}
```

Service层：

````java
class UserService {
  execute() {
    UserDao dao = new UserDao();
    dao.add();
  }
}
````

——注释：

传统的方式，Service层和Dao层耦合度太高，比如说UserDao类的路径发生了变化，那么在Service层里面，也要去改import包的路径

#### b.一般的工厂模式

Dao层：

```java
class UserDao {
  add() {
    ...
  }
}
```

工厂：

```java
class UserFactory {
	public static UserDao getDao() {
		return new UserDao;
	}
}
```

Service层：

```java
class UserService {
  execute() {
    UserDao dao = new UserDao();
    dao.add();
  }
}
```

——注释：

这样比方式a，即传统的直接new对象方式，Service层和Dao层耦合度要低

#### c.IOC过程（即xml解析 + 工厂模式 + 反射）

首先：xml配置文件，配置创建的对象

```xml
<bean id="dao" class="spring5.UserDao"></bean>
```

其次：有了Service类和Dao类，创建工厂（注意，这里不是上面方式b直接new对象，而是用xml解析和反射来操作）

```java
class UserFactory {
  public static UserDao getDao() {
    String classValue = class属性值;// xml解析
    Class clazz = Class.forName(classValue);// 通过反射创建对象
    return (UserDao) clazz.newInstance();
  }
}
```

——注释：

进一步降低耦合度，比如UserDao路径变化了，只要在xml文件中修改即可，后面的层次根本不用担心UserDao路径被修改的事情。

这也就是说，UserDao全部信息都被封装在xml文件中。

### 3、一点小总结

所谓解耦合，就是将信息统一封装在一起（比如说用到的类信息放在.xml配置文件中），然后加上一层。

## 三、IOC接口

### 1、概述

```markdown
IOC思想基于IOC容器完成，IOC容器底层就是对象工厂
```

### 2、Spring提供IOC容器实现的两种方式（两个接口）

#### a.BeanFactory

```markdown
IOC容器基本实现：是Spring内部的使用接口，不提供开发人员进行使用
```

注释：加载配置文件时，不会创建对象，在获取对象（使用）才去创建对象

#### b.ApplicationContext

```markdown
BeanFactory的子接口，提供更多更强大的功能，一般由开发人员进行使用
```

注释：加载配置文件的时候，就会创建配置文件对象

对ApplicationContext接口而言，最重要的两个实现类是：`ClassPathXmlApplicationContext` 和 `FileSystemXmlApplicationContext`

