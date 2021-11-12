# Spring

容器：一种为某种特定组件的运行提供必要支持的一个软件环境。

容器运行组件提供了两个极为强大的功能：`提供组件运行环境`、`提供底层服务`



早先service层如果要使用到dao层，那么必须要new实例化一个dao对象。众所周知，dao层要使用到数据库连接池。这也就说明，每用到一个service层，比如bookService、userService，就要重新连接一个数据库连接池，这样无疑会给系统很大的开销。而且web层会大量使用service层，这些service层也都是new实例化出来的，比方说cartServlet会实例化bookService和userService，historyServlet也会实例化bookService和userService，但是实际上cartServlet和historyServlet可以共享bookService和userService，要是分别创建实例，那么开销也会很大。但是共享实例的话，也不太好处理，到底谁去创建实例，谁又负责获取其他组件已经创建的实例呢？要是再复杂点，依赖关系会更多，要想删除某个组件，其他组件怎么变动？



总结一下核心问题：

1、谁创建组件

2、谁根据依赖关系，组装组件

3、销毁的时候，如何按照依赖顺序正确销毁



那这个时候，IOC就来了。IOC容器，容的都是组件bean。

现在创建组件，我们就不再使用new实例化了，而是使用set方法注入，（构造方法也可以）比方说

```java
public class BookService {
    private DataSource dataSource;

    public void setDataSource(DataSource dataSource) {
        this.dataSource = dataSource;
    }
}
```

直接将DataSource注入到bookService即可，同样DataSource也可以注入到userService。这样就实现了一次实例化，可以反复使用该组件。依赖之间的关系，也可以通过xml配置文件一目了然的看到：（因此IOC也被称作依赖注入）

```xml
<beans>
    <bean id="dataSource" class="HikariDataSource" /><!--创建了dataSource-->
    <bean id="bookService" class="BookService">
        <property name="dataSource" ref="dataSource" /><!--注入了dataSource-->
    </bean>
    <bean id="userService" class="UserService">
        <property name="dataSource" ref="dataSource" />
    </bean>
</beans>
```

一个组件一个bean

调用ApplicationContext（IOC容器接口，通过构造方法，将xml文件放在参数重）的getBean方法，即可创建实例。例如getBean("bookService", BookService.class);



xml文件，将所用到的组件写在一起，这里还不算是创建。由IOC容器接口的实现类（将xml文件作为参数）统一创建实例，这样bean类修改了，和你上层没有关系，只要在xml文件中改就可以了，真正实现了解耦合。



通过xml文件，组件之间的逻辑关系，一目了然。

但是xml文件写起来太繁琐了，所以直接用注解。

```java
@Component
public class MailService {
    ...
}
```

```java
@Component
public class UserService {
    @Autowired
    MailService mailService;

    ...
}
```

```java
@Configuration
@ComponentScan
public class AppConfig {
    public static void main(String[] args) {
        ApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        UserService userService = context.getBean(UserService.class);
        User user = userService.login("bob@example.com", "password");
        System.out.println(user.getName());
    }
}
```

`@Configuration`表示是配置类，`@ComponentScan`表示在该类所在的包以及其子包下的`@component`都可以是它创建的bean，并根据`@Autowired`进行装配（就是扫描的意思）。

