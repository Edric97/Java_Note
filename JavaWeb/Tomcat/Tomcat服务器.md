## 一、服务器（不管软的还是硬的），是要对外暴露资源的！

软件概念的服务器：

```markdown
指的是硬件服务器上被动参与通讯的进程
```

——注释：有被动就有主动，主动的进程是客户机上发起通讯的进程。那这样来说，访问软件服务器，一定要写上硬件服务器上的特定的端口号（即为软件服务器的端口）。



以Tomcat为例，Tomcat占用的是8080端口，他人要想访问我发布的这个项目。除了要写上我主机的ip地址，还要加上8080端口，然后在8080端口下，找路径资源。



## 二、Tomcat服务器

Tomcat包含了三个重要的组成：

* 处理html，js，css等静态页面的Web容器（这里是继承了Apache作为Web服务器的基本功能）
* 根据不同的请求来调用不同的servlet的servlet容器Catalina（因为更细节点说，tomcat作为servlet容器，实际上是tomcat的catalina部分是servlet容器。而这也是tomcat最核心的部分）
* 编译jsp的引擎Jasper



Tomcat的设计是基于模块化设计的，内部主要依赖于不同的模块组件构成。

```markdown
一个Tomcat只有一个server作为根，它管理着多个service服务，而service服务又管理着多个Connector以及一个Container，其中核心组件就是Connector以及Container
```

解释：

* Server组件：一个Tomcat只能有一个Server，Server就是一个Tomcat实例
* Service组件：其实是一个集合，将Connector组件与Container组件包装在一起，对外进行服务
* Connector（连接器）组件：主要负责`监听`指定端口的 `客户端请求 ` （不同端口对应不同Connect组件，因为发起请求的客户端很多，所以说要有多个Connector组件），将Socket请求过来的数据，都封装成Request请求对象，同时将该请求对象传递给Container容器进行下一步处理。因此其作用就是——将客户端的请求转发到Container容器，这也是为什么被称作连接器的原因。
* Container组件：该组件是最接近Web应用的组件，它负责根据请求进行一系列的servlet调用。本身又包含四个子容器：`Engine, Host, Context, Wrapper`



## 三、一个完整的流程

1、当我们浏览器点击事件发生，发送了一个http/https的请求，首先到达tomcat，即运行的实例server中

2、该请求被监听 8080 端口的 connector监听到，获取请求报文后，封装成Request请求，并将该请求发往Engine

3、Engine根据请求的url，搜寻使用哪个Host

4、当相应的Host获取该请求后，根据请求中的地址，找寻相应的Context来处理该请求

5、Context根据其内部的映射表，获取相应的servlet，并构造`HttpServletRequest`对象和`HttpServletResponse`对象，进行业务处理

6、Context将处理完的`HttpServletResponse`对象返回给`Host`

7、Host再将结果返回Engine

8、Engine中心调度，将结果返回给 connector

9、connector将结果返归给客户端



## 四、一张图——访问百度的完整流程

虽然不是用Tomcat，但是还是能很清晰的看出软件服务器是如何运作的！

![访问百度的完整流程](/Users/caoxiaodong/Desktop/访问百度的完整流程.jpeg)



