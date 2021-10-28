# Bean管理

## 一、Bean的两种类型

### 1、普通Bean类型

特点是：定义类型和返回类型一定是一致的

就是最平常的Bean类，比如User类，Student类这种

### 2、工厂Bean类型

特点是：定义类型和返回类型不一定是一致的

## 二、工厂Bean类型

前面说了很多的普通的Bean类型，这里我们重点谈谈工厂Bean类型。

怎么创建工厂Bean类型呢？

step1：创建类，让这个类作为工厂Bean，实现接口FactoryBean

step2：实现接口里面的方法，在实现的方法中定义返回的Bean类型

```java
public class MyBean implements FactoryBean<Course> {
  @Override
  public Course getObject() throws Exception {// 通过泛型机制，xml文件的返回类型就不是MyBean，而是Course类
    
  }
  @Override
  ...
    
  @Override
  ...
}
```

## 三、IOC下Bean的作用域

要在Spring里面，设置创建的Bean实例，是单实例还是多实例。如果没有设置的话，默认是单实例。

——注释：

单实例就是用同一个工厂<bean>生产出来的实例，都指向堆空间中的同一个对象。多实例，则指向堆空间的不同对象。

简单来说，看对象的地址是否相同即可，相同则是单实例，否则是多实例。

例如：

```java
Book book1 = context.getBean("book", Book.class);
Book book2 = context.getBean("book", Book.class);

/**
book1 和 book2 的地址相同，也就是说从同一个xml文件中实例化的对象是同一个
*/
```

那么怎么设置多实例呢？

很简单，只要在<bean>标签里面，加上`scope`属性即可：

```xml
<bean id="book" class="spring5.Book" scope="prototype"></bean>
```

那么单实例呢？

也很简单，要么不写`scope`属性，要么设置成`singleton`：

```xml
<bean id="book" class="spring5.Book" scope="singleton"></bean>
```

## 四、Bean生命周期

所谓生命周期，就是指从对象创建到对象销毁的过程。

### 1、Bean的生命周期

```markdown
1、通过构造器创建Bean实例（无参构造）
2、为Bean的属性设置值和其他Bean引用（调用set方法）
3、调用Bean的初始化方法（需要进行配置该初始化方法——即，在<bean>标签里添加init-method属性）
4、Bean可以使用了（对象获取到了）
5、当容器关闭的时候，调用Bean销毁方法（需要配置销毁方法——即，在<bean>标签里添加destroy-method属性）
```

### 2、Bean的后置处理器

所谓的Bean的后置处理器，实际上就是个类，不过实现了BeanPostProcessor接口。

这样在xml文件配置的时候，Spring就会为这个xml文件下，所有其他类绑定该后置处理器。

具体上来说有两步：

```markdown
1、把Bean实例传递给Bean后置处理器的方法PostProcessBeforeInitialization —— 这一步在上面的2，3步之间
2、把Bean实例传递给Bean后置处理器的方法PostProcessAfterInitialization —— 这一步在上面的3，4步之间
```

因为，Bean的生命周期严格上来说，是有7步的。

