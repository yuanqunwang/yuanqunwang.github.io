# Spring Web MVC

## 配置Spring MVC框架

### 基于XML文件配置

Servlet 部署描述文件的配置（`/webapp/WEB-INF/web.xml`）：

```xml
<context-param>
  <param-name>contextConfigLocation</param-name>
  <param-value>classpath:application-*.xml</param-value>
</context-param>

<listener>
  <listener-class>org.springframework.web.context.ContextLoaderListener</listener-class>
</listener>

<servlet>
  <servlet-name>mydispatch</servlet-name>
  <servlet-class>org.springframework.web.servlet.DispatcherServlet</servlet-class>
  <init-param>
    <param-name>contextConfigLocation</param-name>
    <param-value>classpath:spring-mvc.xml</param-value>
  </init-param>
  <load-on-startup>1</load-on-startup>
</servlet>
<servlet-mappine>
  <servlet-name>mydispatch</servlet-name>
  <url-pattern>/</url-pattern>
</servlet-mappine>
```

配置文件中：

* listener 在spring容器启动时，读取 context-param 节点定义的spring xml 配置文件中定义的beans，这些beans 主要是 Dao, Service 类。作为Spring mvc 的 `WebApplicationContext` 的父容器。

* Spring mvc 的核心是 `DispatcherServlet`， 作为mvc框架的前端控制器。其本质是一个`Servlet`，所以就是一个`Servlet` 的配置。

*  init-param 配置的是启动 `WebApplicationContext`容器所需的xml文件。如果没有配置项，则 `DispatcherServlet`按照约定，去读取`mydispatch-servlet.xml`的配置文件，其中`mydispatch`是servlet-name。

* `load-on-startup`元素定义Servlet的加载顺序，当该元素未定义或者元素值为负数，则Servlet 容器确定何时加载该Servlet；当该元素的值为大于等于0的正数时，Servlet的加载顺序与元素值负相关，即值越小，加载顺序在前，反之，亦然。

* `mydispatch-servlet.xml`文件中主要配置项是组成Spring mvc 各个部件的bean，示例配置如下：

  ```xml
  <!-- 注解驱动 -->
  <mvc:annotation-driven/>
  <context:component-scan base-package="com.example.project.controller"/>
  
  <!-- 视图解析器 -->
  <bean id="viewResolver" class="org.springframework.web.servlet.view.InternalResourceViewResolver">
    <property name="prefix" value="/WEB-INF/web/"/>
    <property name="suffix" value=".jsp"/>
  </bean>
  
  <!-- 拦截器 -->
  <mvc:interceptors>
    <mvc:interceptor>
      <mvc:mapping path="/path/to/be/interceptor"/>
      <bean class="com.example.MyInterceptor"/>
    </mvc:interceptor>
  </mvc:interceptors>
  
  <!-- 文件上传解析器 -->
  <bean id="multipartResolver" class="org.springframework.web.multipart.CommonsMultipartResolver">
    <property name="maxUploadSize" value="10000"/>
    <property name="defaultEncoding" value="utf-8"/>
  </bean>
  ```

  在上述文件中定义的special bean 会取代spring mvc 框架默认的的配置，spring mvc 的默认special bean 的默认配置在 `DispatcherServlet.properites` 中定义。

### 基于Java类配置

* `DispatcherServlet`的配置

  ```java
  public class WebServletConfiguration implements WebApplicationInitializer {
    public void onStartup(ServletContext ctx) throw ServletException {
      AnnotationConfigWebApplicationContext webCtx = AnnotationConfigWebApplicationcontext();
      webCtx.register(SpringConfig.class);
      webCtx.setServletcontext(ctx);
      ServletRegistration.Dynamic servlet = ctx.addServlet("dispatcher", new DispatcherServlet(webCtx));
      servlet.setLoadOnStartup(1);
      servlet.addMapping("/");
    }
  }
  ```

* `WebApplicationContext`的配置，定义special bean，相当与`mydispatch-servlet.xml`的配置。

  ```java
  @EnableWebMvc
  @ComponentScan(basePackages="com.example.project.controller")
  public class SpringConfig extends WebMvcConfigurerAdapter {
    @Bean
    public ViewResolver viewResolver() {
      InternalResourceViewResolver viewResolver = new InternalResourceViewResolver();
      viewResolver.setViewClass(JstlView.class);
      viewResolver.setPrefix("/WEB-INF/pages");
      viewResolver.setSuffix(".jsp");
      return viewResolver;
    }
    
    @Override
    public void configureDefaultServletHanding(DefaultServletHandlerConfigurer configurer){
      configurer.enable();
    } 
  }
  ```

## MVC框架的功能

* 将某个http请求

## 总体架构

<img src="assets/images/RequestLifecycle.png" style="zoom:75%;" />

## DispatcherServlet的类继承体系

![](assets/images/dispather-servlet-arc.png)

* `GenericServlet`是一个通用的 Servlet 抽象类，定义了一个 service 方法。

* `HttpServlet` 是针对 Http 协议的抽象类，service 方法中，根据Http请求的方法分发到 `doGet`, `doPost`等方法中去。

* `HttpServletBean`是第一个感知到Spring的类，主要作用是注入init-param参数，重写Servlet的`init`方法，同时，该方法调用 `initServletBean`方法，最终由`FrameworkServlet`重写。

* `FrameworkServlet`作用

  * 根据Servlet的init-param参数，创建`WebApplicationContext`，如果init-param参数中指定了 contextClass，则使用该contextClass创建容器，否则使用默认的`XmlWebApplicationContext`创建容器。此外，实现了`ApplicationContextAware`接口，可以通过外部注入Web容器。如果是外部注入的容器，通过`webApplicationContextInjected=true`来防止外部注入的容器被关闭。
  * 将`HttpServlet`的`doGet`, `doPost`等方法统一分配给`processRequest`方法，最终调用抽象方法`doService(HttpServletRequest, HttpServletResponse)`。
  * 通过Java的方式配置

* `DispatcherServlet`类

  * Model-View-Controller之间的交互控制。

  * 将Http请求，映射到合适的请求处理器。

  * 将Http请求中的各种参数，统一转换成请求处理器方法的签名参数对象。

  * 请求处理方法返回`ModelAndView`对象时，通过视图解析器解析响应的视图`View`，并将视图结合模型渲染返回。

  * 重写`FrameworkServlet`类的`doService`方法，将请求的处理进一步分发给`doDispatch(HttpServletRequest, HttpServletResponse)`。

  * `doService`方法将`WebApplicationContext`中的一些special bean设置为request属性，方便请求处理：

    ```java
    request.setAttribute(WEB_APPLICATION_CONTEXT_ATTRIBUTE, getWebApplicationContext());
    request.setAttribute(LOCALE_RESOLVER_ATTRIBUTE, this.localeResolver);
    request.setAttribute(THEME_RESOLVER_ATTRIBUTE, this.themeResolver);
    request.setAttribute(THEME_SOURCE_ATTRIBUTE, getThemeSource());
    ```

## DispatcherServlet的启动过程

```java
public abstract class HttpServletBean extends GenericServlet{
  @Override
  public final void init() {
    //属性注入，检查操作
    
    initServletBean();
  }
  protected abstarct void initServletBean();
}

public abstract class FrameworkServlet extends HttpServletBean {
  @Override
  protected void initServletBean() {
    // 其他操作
    
    onRefresh();
  }
  
  protected abstract void onRefresh(Applicationcontext);
}

public class DispatcherServlet extends FrameworkServlet {
  @Override
  protected void onRefresh(ApplicationContext ac) {
    this.initStrategies(ac);
  }
  
  private void initStrategies(ac) {
    this.initMultipartResolver(ac);
    this.initLocaleResolver(ac);
    this.initThtmeResolver(ac);
    this.initHandlerMappings(ac);
    this.initHandlerAdapters(ac);
    this.initHandlerExceptionResolver(ac);
    this.initRequestToViewNameTranslator(ac);
    this.initViewResolver(ac);
    this.initFlashMapManager(ac);
  }
}
```

## http请求的处理过程

Http请求到达`DispatcherServlet`后，选出合适的请求处理对象，这个确定请求处理对象的工作有`HandlerMapping`完成。同时对请求处理对象没有限制，不需要实现特定的接口，所以需要一个Adapter统一接口的工作。

### `HanlderMapping`

`HandlerMapping`接口的主要作用是，将Http请求映射到请求处理对象。该接口的声明如下：

```java
public interface HandlerMapping {
  HandlerExecutionChain getHandler(HttpServeltRequest) thouws Exception;
}
```

### `HandlerAdapter`

`HandlerAdapter`为适配器模式的使用。用于统一调用请求处理对象的接口，这样保证对任何请求处理对象的调用都会返回相同的`ModelAndView`对象。该接口的定义如下：

```java
public interface HandlerAdapter {
  @Nullable
  ModelAndView handle(HttpServletRequest, HttpServletResponse, Object handler) thows Exception;
}
```

`HandlerAdapter#handle` 方法在请求处理对象完成请求后，可以有以下两种处理形式：

* 直接将处理结果返回给响应对象，返回`Null`，这种处理方式适用于RESTful风格的网络服务。
* 返回一个`ModelAndView`对象。适用于传统的MVC网络应用。

最常使用的实现类`RequestMappingHandlerAdapter`，将`HttpServletRequest`请求对象中的参数转化为请求方法的入参对象，同时可以根据需要，将请求处理方法返回的`String`转换为对应的`ModelAndView`对象，或者将请求处理方法返回的对象转换成Json、xml格式的数据返回。实现上述功能的的代码：

```java
ServletInvocableHandlerMethod invocableMethod 
  = createInvocableHandlerMethod(handlerMethod);
if (this.argumentResolvers != null) {
    invocableMethod.setHandlerMethodArgumentResolvers(
      this.argumentResolvers);
}
if (this.returnValueHandlers != null) {
    invocableMethod.setHandlerMethodReturnValueHandlers(
      this.returnValueHandlers);
}
```

其中将请求参数转换成处理方法的入参接口：`HandlerMethodArgumentResolver`，该接口有接近30多个实现类。

将请求处理方法返回值处理成`ModelAndView`对象，或json等格式的数据的接口：`HandlerMethodReturnValueHandler`

### `ViewResolver`

`ViewResolver`根据`ModelAndView`对象，找出合适的视图对象。

```java
for (ViewResolver viewResolver : this.viewResolvers) {
    View view = viewResolver.resolveViewName(viewName, locale);
    if (view != null) {
        return view;
    }
}
```

### `View`

通过调用`View`对象的`render`方法，将html页面返回给客户端。

## `@RequestMapping`

* `@RequestMapping` 注解有类级别和方法级别的，如果在类级别标注了该注解，则必须添加该注解地址到访问的url中去，比如：

  ```java
  @Controller
  @RequestMapping("test")
  public class MyController {
  	@RequestMapping("/hello/*")
  	public String hello() {
  		return "hello";
  	}
  }
  ```

  则访问的url为`.../test/hello/` 才会映射到`hello` 方法。

## 使用到的设计模式

* MVC模式
* 策略模式：各种解析器，如`ViewResolver`, `LocalResolver`等。
* 适配器模式：`HandlerAdapter`
* 简单工厂方法模式，如`PropertyAccessorFactory`。



参考资料：

* [how-spring-mvc-really-works](https://dzone.com/articles/how-spring-mvc-really-works)

