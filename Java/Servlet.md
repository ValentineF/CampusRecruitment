# Servlet

## 本质

Servelt的本质就是一个Java的接口，提供了一个规范的Java Web应用的接口，使得其他的主流Java Web技术之间有较好的互通性

## Servlet源码

```java
public interface Servlet {

    public void init(ServletConfig config) throws ServletException;

    public ServletConfig getServletConfig();

    public void service(ServletRequest req, ServletResponse res)throws ServletException, IOException;

    public String getServletInfo();

    public void destroy();
}
```

## Tomcat和Servelt

Servlet本身不处理任何请求，比如servlet不会去监听特定的请求端口或者直接和客户端通信，不管理任何任何资源。相反，这些由servlet的容器处理，典型如Tomcat()，所以servlet能够应用于不同的环境。

## Servlet的生命周期

1. Tomcat接受客户端请求
2. Tomcat根据请求映射到对应的处理引擎
3. 当请求被映射到一个正确的servlet
    - 若该servlet没有被加载，Tomcat编译该servlet的字节码，并让JVM运行来创建一个servlet对象
4. Tomcat初始化servlet后，调用servlet的init()方法。
    - servlet读取Tomcat的资源配置信息，由Tomcat依次创建所需要的资源
5. servlet初始化完成，Tomcat调用servlet的service()方法来处理本次请求，返回response
6. 在servlet的生命周期中，Tomcat能监听并保存servlet的状态，并且允许其他servlet获取这些信息。这些信息将会被持久化保存，session或者其他上下文组件会利用这些信息，做特定的操作。例如购物车。
7. Tomcat调用servlet的destroy()方法，销毁servlet。
8. ![Servlet的请求过程](../picture/servlet.jpg)

## 参考资料

- [tomcat-servlet](https://www.mulesoft.com/cn/tcat/tomcat-servlet)
