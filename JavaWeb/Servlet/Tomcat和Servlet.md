# Tomcat 和 Servlet

## 一、Tomcat

## 二、Servlet

### 1、看一下名字Servlet

类似于Server Applet，Applet中文为 小程序

### 2、一些关系：

先看一个Servlet程序：

```markdown
public abstract class BaseServlet extends HttpServlet {
    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        doPost(req, resp);
    }

    @Override
    protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {

        String action = req.getParameter("action");

        try {
            Method method = this.getClass().getDeclaredMethod(action, HttpServletRequest.class, HttpServletResponse.class);//this指向的是子类实例
            method.invoke(this, req, resp);
        } catch (NoSuchMethodException e) {
            e.printStackTrace();
        } catch (IllegalAccessException e) {
            e.printStackTrace();
        } catch (InvocationTargetException e) {
            e.printStackTrace();
        }

    }
}
```



其中`HttpServlet`是继承自`GenericServlet`的，而`GenericServlet`实现了三个接口——`Servlet, ServletConfig, Serializable`，其中我们着重看`Servlet`接口，该接口定义了五个方法：



```markdown
public interface Servlet {
    void init(ServletConfig var1) throws ServletException;

    ServletConfig getServletConfig();

    void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;

    String getServletInfo();

    void destroy();
}
```



其中`init(),service(),destroy()`方法，说明了一个Servlet程序的生命周期。

——现在看一下最重要的`service()`方法，其给定的参数是`ServletRequest var1, ServletResponse var2`，分别是Servlet处理请求（`ServletRequest`）和返回响应（`ServletResponse`）的，其中`ServletRequest`是接口，里面定义了处理请求的一堆方法，包括了常用的`getParameter(String var1)`。

——而最上面那一大段的Servlet程序，里面的doGet 和 doPost方法，里面的参数都是`HttpServletRequest req, HttpServletResponse resp`，而接口`HttpServletRequest, HttpServletResponse`这两个分别是接口`ServletRequest`和`ServletResponse`的子接口。

————》HttpServlet 是 Servlet最重要的实现类，虽然没有抽象方法，但是是抽象类，因为不想被实例化

```markdown
继承关系：
HttpServlet(C) ---> GenericServlet(C) ---> Servlet(I)

Servlet(I) ---> init(),service(ServletRequest var1, ServletResponse var2),destroy

HttpServletRequest(I) ---> ServletRequest(I)
HttpServletResponse(I) ---> ServletResponse(I)
```



————》看一下service方法：

```
Servlet接口：
	void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
		
GenericServlet类：
	public abstract void service(ServletRequest var1, ServletResponse var2) throws ServletException, IOException;
		
HttpServlet类：（有方法的重载）
	1、public void service(ServletRequest req, ServletResponse res) throws ServletException, IOException {
        HttpServletRequest request;
        HttpServletResponse response;
        try {
            request = (HttpServletRequest)req;
            response = (HttpServletResponse)res;
        } catch (ClassCastException var6) {
            throw new ServletException(lStrings.getString("http.non_http"));
        }

        this.service(request, response);
    }
  2、protected void service(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
  
        String method = req.getMethod();
        /**
        getMethod()这个方法，在HttpServletRequest接口中才写的，在ServletRequest是没有的，但是具体实现是在 public class HttpServletRequestWrapper extends ServletRequestWrapper implements HttpServletRequest，即 HttpServletRequestWrapper 类中实现的。
        */
        
        long lastModified;
        if (method.equals("GET")) {
            lastModified = this.getLastModified(req);
            if (lastModified == -1L) {
                this.doGet(req, resp);
            } else {
                long ifModifiedSince;
                try {
                    ifModifiedSince = req.getDateHeader("If-Modified-Since");
                } catch (IllegalArgumentException var9) {
                    ifModifiedSince = -1L;
                }

                if (ifModifiedSince < lastModified / 1000L * 1000L) {
                    this.maybeSetLastModified(resp, lastModified);
                    this.doGet(req, resp);
                } else {
                    resp.setStatus(304);
                }
            }
        } else if (method.equals("HEAD")) {
            lastModified = this.getLastModified(req);
            this.maybeSetLastModified(resp, lastModified);
            this.doHead(req, resp);
        } else if (method.equals("POST")) {
            this.doPost(req, resp);
        } else if (method.equals("PUT")) {
            this.doPut(req, resp);
        } else if (method.equals("DELETE")) {
            this.doDelete(req, resp);
        } else if (method.equals("OPTIONS")) {
            this.doOptions(req, resp);
        } else if (method.equals("TRACE")) {
            this.doTrace(req, resp);
        } else {
            String errMsg = lStrings.getString("http.method_not_implemented");
            Object[] errArgs = new Object[]{method};
            errMsg = MessageFormat.format(errMsg, errArgs);
            resp.sendError(501, errMsg);
        }

    }
```



现在我们再来看一下，具体的Tomcat和Servlet是如何运转的：

```markdown
1、Web Client 向 Servlet容器（Tomcat）发送HTTP请求
2、Servlet容器接收Web Client的请求
3、Servlet容器创建一个HttpRequest对象，将Web Client请求的信息封装到这个对象中
4、Servlet容器创建一个HttpResponse对象（参数对象优先生成）
5、Servlet容器调用HttpServlet对象的service()方法，把上面3、4步创建的 HttpRequest 对象和 HttpResponse 对象作为参数传给HttpS ervlet对象
6、HttpServlet调用HttpRequest对象的有关方法（一般指的是doGet或者doPost方法），获取Http请求信息
7、HttpServlet调用HttpResponse对象的有关方法，生成响应数据
8、Servlet容器把HttpServlet的响应结果传递给Web Client
```

关于上面第6、7步的具体细节，可以看上面`HttpServlet`类重载的`service()`方法，得看具体的请求是什么类型的，如果是GET类型的，就会调用`HttpServlet`的`doGet()`方法。但是`doGet()`和`doPost()`方法，在`HttpServlet`中，都长的像下面这个样子：

```markdown
protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String msg = lStrings.getString("http.method_get_not_supported");
        this.sendMethodNotAllowed(req, resp, msg);
}

protected void doPost(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        String msg = lStrings.getString("http.method_post_not_supported");
        this.sendMethodNotAllowed(req, resp, msg);
}
```

所以这两个方法，都需要在自己的业务逻辑的Servlet类（继承HttpServlet）中重写。



问题1:为什么要继承HttpServlet，而不是GenericServlet呢？

因为：虽然GenericServlet只有一个抽象的方法，即`service()`方法，但是我们就是要重写`service()`，这就很麻烦了。因为我们要自己实现对各种类型的请求的覆盖，比方说get请求。但是HttpServlet类中，这些全部都做好了，继承HttpServlet只要重写`doGet()`等即可实现业务逻辑。



问题2:Servlet创建的时机？

在Servlet容器启动的时候：读取web.xml配置文件中的信息，构造指定的Servlet对象，创建ServletConfig对象（这个一般还是HttpServlet对象，因为HttpServlet继承自GenericServlet类，而GenericServlet实现了ServletConfig接口，并且对ServletConfig接口中的方法进行了比较好的实现。因此虽然说上面的ServletConfig对象指的是HttpServlet对象，但是HttpServlet类中却没有写ServletConfig接口中的方法，这些都在GenericServlet中写过了），同时将ServletConfig对象作为参数来调用Servlet对象的init方法。





参考：

https://blog.csdn.net/danielzhou888/article/details/70835418

https://geek-docs.com/servlet/servlet-tutorial/genericservlet-class.html





简单复习一下面向接口编程：

`HttpServlet`重载的`service()`方法中，扔进去的是两个接口，直接调用了接口的某个方法。这个就是面向接口编程，在具体的实现的时候，即main方法中，说明是哪个实现类就好了。例如如下代码：

```markdown
public class Test1 {

    public static void main(String[] args) {
        A a = new C();
        testInterface(a);
    }

    public static void testInterface(A a) {
        a.test();
    }
}

interface A {
    void test();
}

class B implements A {
    public void test() {
        System.out.println("测试面向接口编程1");
    }
}
class C implements A {
    @Override
    public void test() {
        System.out.println("测试面向接口编程2");
    }
}
```







