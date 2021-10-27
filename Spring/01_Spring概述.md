# Spring概述

## 一、概述

首先让我们对Spring有个初步的理解，即Spring是什么，能干什么，是什么能Spring这么能干？

### 1、Spring是什么

```markdown
Spring是轻量级的开源的JavaEE框架
```

注：所谓的“轻量级”就是指，所依赖的jar包不是很多。“开源”在GitHub上，但是GitHub上只有源码，没有可下载的jar（还需要另费一番周折才能下载到）。

### 2、Spring能干什么

```markdown
Spring能解决企业开发的复杂性
```

### 3、Spring为什么这么强

因为Spring具有以下的特点：

```markdown
1、方便解耦（这里用到了IOC），简化开发
2、AOP（不修改源代码进行功能增强）编程支持
3、方便程序测试（JUnit框架）
4、方便和其他框架进行整合
5、方便进行事务操作
6、降低API开发难度
```

之所以注明 IOC（控制反转） 和 AOP（面向切面），是因为这两点在Spring中非常重要。

## 二、如何下载Spring框架

```markdown
1、进入Spring官网，选择Project，选择SpringFramework
2、点击GitHub的图标，进入GitHub链接
3、在GitHub页面中，一直往下翻，找到黑体的标题名——Access to Binaries，点击链接 Spring Framework Artifacts
4、在此GitHub页面下，找到黑体标题名——Spring Repositories，点击链接 https://repo.spring.io 
5、进入Spring仓库，点击左边标签栏的Artifacts
6、找到该页面左边子菜单下的 libs-release-local | org | springframework | 具体的Spring版本（这里选择的是5.2.6，文件后缀名dist.zip）
```

## 三、一个简单的小示例

```markdown
1、首先新建一个模块
2、在模块下，创建一个lib用来存放用到的jar包
	这里用到的jar包是：spring-framework-5.2.6.RELEASE | libs | spring-beans-5.2.6.RELEASE.jar && spring-context-5.2.6.RELEASE.jar && spring-core-5.2.6.RELEASE.jar && spring-expression-5.2.6.RELEASE.jar
	当然还有个commons-logging包
3、在src下写源码，然后在src下创建一个配置文件，配置刚才写的源码
4、在src创建一个测试包，并在测试包下创建测试类
```

下面简单写一下源代码的内容：

```java
package spring5;

public class User {
    public void add() {
        System.out.println("Hello, Spring");
    }
}
```

下面简单写一下配置文件的内容：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="spring5.User"></bean>
</beans>
```

下面简单写一下测试类的方法：

```java
package spring5.test;

import org.junit.Test;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import spring5.User;

public class TestUser {
    @Test
    public void testAdd() {
        ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");

        User user = context.getBean("user", User.class);

        System.out.println(user);
        user.add();
    }
}
```

注：

1、上面测试文件中代码：

```java
ApplicationContext context = new ClassPathXmlApplicationContext("bean1.xml");
```

就是加载Spring配置文件

2、上面测试文件中代码：

```java
User user = context.getBean("user", User.class);
```

则是获取配置文件创建的对象