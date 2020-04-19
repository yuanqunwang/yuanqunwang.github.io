# IoC容器

## 一般概念

IoC容器是Spring框架的核心，通过Java的反射（refection）机制配置和管理Java对象。容器负责管理Java Bean的生命周期：创建对象，调用对象的初始化方法，对象的销毁等。

通过IoC容器创建的对象称为managed objects或beans。

Spring容器的优势：

* 降低对象之间的耦合度
* 容器管理Bean的生命周期
* 方便测试

容器通过结合可配置的元数据和Java POJO类，实现对象的创建、配置以及组装等，可配置的元数据可以是以下文件形式：

 - XML文件
 - Java注解
 - Java类
 - Groovy DSL

下图提供更高的视角解释Spring容器的工作：

![](assets/images/spring_ioc_container.png)

> 指定bean的名称，实现类，依赖注入，bean的生命周期管理，bean的作用域，是否延迟加载6个方面控制一个bean，其中依赖注入和bean的生命周期管理是重点。4种配置bean的方式，只是通过不同形式提供与bean有关的以上6种信息。

----------

## Spring中提供的容器

BeanFactory是Spring框架的基础设施，面向Spring本身；ApplicationContext面向Spring框架的开发者，几乎所有的应用场合都可以直接使用ApplicationContext而非底层的BeanFactory。在移动设备或基于applet的应用使用BeanFactory。

### BeanFactory

BeanFactory的类继承体系如下：

![](assets/images/beanfactory.png)

使用BeanFactory创建bean对象的实例：

```java
public class BeanFactoryTest{
    @Test
    public void getBean() throws Throwable{
        ResourcePatternResolver resolver = new PathMatchingResourcePatternResolver();
        Resource res = resolver.getResource("classpath:beans.xml");
        DefaultListableBeanFactory factory = new DefaultListableBeanFactory();
        XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(factory);
        reader.loadBeanDefinitions(res);
        Car car = factory.getBean("car", Car.class);
    }
}
```



### ApplicationContext

![](assets/images/application_context.png)

### WebApplicationContext

![](assets/images/web_application_context.png)



## Bean的生命周期

### BeanFactory容器中bean的生命周期

当调用容器的`getBean()`方法时，bean在容器中会经历下图所示的一系列过程：

![](assets/images/bean_lifecycle_beanfactory.png)

* 如果某个bean配置为`singleton`，则完成`postProcessAfterInitialization()`方法之后，容器继续管理bean的生命周期，且将其放置于Spring的缓存池中，下次再次调用时，直接从缓存池获取，而不必经历之前的一系列生命周期的处理。
* 如果bean配置为`prototype`，则完成`postProcessAfterInitialization()`方法之后，容器直接将bean返回给调用者，如果再次调用，则经历相同的一系列生命周期的处理。
* bean的完整生命周期方法的调用大致分为如下4类：
  1. Bean自身的方法，Bean的构造方法、setter方法、init-method以及destroy-method；
  2. Bean级生命接口方法，如`BeanNameAware`,`BeanFactoryAware`,`InitalizingBean`和`DisposableBean`接口方法；
  3. 容器级生命周期几口方法，如`InstantiationAwareBeanPostProcessor`和`BeanPostProcessor`接口方法，一般将其实现类称为“后处理器”。Spring容器创建任何Bean时，这些后处理器都会发生作用。
  4. 工厂后处理器接口方法，包括`AspectJWeavingEnabler`,`CustomAutowireConfigurer`以及`ConfigurationClassPostProcessor`，工厂后处理器也是容器级的，在应用上下文装配配置文件后立即调用。

### ApplicationContext容器中bean的生命周期

与BeanFactory中的生命周期类似，但有以下几个不同点：

* 在ApplicationContext容器中，如果Bean实现类`org.springframework.context.ApplicationContextAware`接口，则会增加一个调用该接口方法`setApplicationContext()`方法的步骤；
* ApplicationContext容器中，只需在配置文件中通过bean定义工厂后处理器，容器会利用Java反射机制自动识别出配置文件中定义的`BeanPostProcessor`,`InstantiationAwareBeanPostProcessor`和`BeanFactoryPostProcessor`，并将它们注册到应用上下文中，而BeanFactory中需要手动调用`addBeanPostProcessor()`方法，这也是在应用开发时普遍使用ApplicationContext的原因之一；

在配置文件中配置了`BeanFactoryPostProcessor`接口的实现类，则在容器装载配置文件之后，初始化Bean实例之前，调用这些工厂后处理器对配置信息进行加工处理，Spring框架提供了多个工厂后处理器，如：`CustomEditorConfigurer`,`PropertyPlaceholderConfigurer`等，如果在配置文件中配置了多个工厂后处理器，最好实现`org.springframework.core.Ordered`接口，以便Spring按确定的顺序调用它们。

![](assets/images/bean_lifecycle_applicationcontext.png)

## 在IoC容器中装配Bean

通过加载xml文件或检测Java注解的方式来配置IoC容器，xml文件或Java注解中包含了创建bean所必须的信息。

![](assets/images/springcontainer.png)

## 依赖注入（Dependency Injection）

* Dependency Lookup：调用者通过调用`getBean()`方法，获得容器中特定的bean，一般传递给`getBean()`的参数有bean的名称、bean的类型。
* 依赖注入（Dependency Injection）：容器根据bean的名称，将一个bean传给另外一个bean。一般有3种方式，属性注入、构造方法注入和工厂方法注入。

### 属性注入

是指通过setter方法注入bean的属性值或依赖对象。要求类有一个无参构造方法和对应属性的setter方法（但不要求setter方法有对应的属性），Spring首先通过调用Bean的默认构造方法实例化对象，然后通过反射的方法调用对应的setter方法注入属性值。

```xml
<bean id="user" class="com.philowong.dao.UserDaoImpl"></bean>
<bean id="service" class="com.philowong.service.UserServiceImpl">
   <property name="userDao" ref="user"></property>
</bean>
```

### 构造方法注入

Spring在实例化Bean时就注入属性，确保Bean在实例化之后的可用性。

```xml
<bean id="user" class="com.philowong.dao.UserImpl"></bean>
<bean id="service" class="com.philowong.service.UserServiceImpl">
   <constructor-args index="0" ref="user"></construct-args>
</bean>
```

### 工厂方法注入

#### 非静态工厂方法

```java
package com.philowong.CarFactory;

//非静态方法
public class CarFactory{
    public Car createBMWCar(){
        Car car = new Car();
        car.setBrand("BMW");
        return car;
    }
}
```

```xml
<bean id="carFactory" class="com.philowong.CarFactory"></bean>
<bean id="bmwCar" factory-bean="carFactory" factory-method="createBMWCar"></bean>
```

#### 静态工厂方法

```java
package com.philowong.CarFactory;

//静态方法
public class CarFactory{
    public static Car createBMWCar(){
        Car car = new Car();
        car.setBrand("BMW");
        return car;
    }
}
```

```xml
<bean id="bmwCar" class="com.philowong.CarFactory" factory-method="createBMWCar"></bean>
```

## 注入参数详解

* 字面值

  ```xml
  <bean id="car" class="com.smart.attr.Car">
      <property name="brand">
      	<value><![CDATA[BMW&7]]</value>
  	</property>
  </bean>
  ```

* 引用其它bean

  ```xml
  <bean id="car" class="com.smart.attr.Car"></bean>
  <bean id="boss" class="com.smart.attr.Boss">
      <property name="car">
      	<ref bean="car"></ref>
  	</property>
  </bean>
  ```

* 内部bean

  ```xml
  <bean id="boss" class="com.smart.attr.Boss">
      <property name="car">
          <bean class="com.smart.attr.Car">
              <property name="maxSpeed" value="200"/>
              <property name="price" value="200000.0"/>
          </bean>
      </property>
  </bean>
  ```

  

* null值

  ```xml
  <property name="brand"><null/></property>
  ```

* 级联属性

  ```xml
  <bean id="boss" class="com.smart.attr.Boss">
      <property name="car.brand" vlaue="BMW"/>
  </bean>
  ```

* 集合类属性

  ```xml
  <!-- 配置list、set属性 -->
  <property name="favorites">
      <list><!-- set集合相同-->
          <value>book</value>
          <value>music</value>
      </list>
  </property>
  <!-- 配置map属性 -->
  <property name="jobs">
      <map>
          <entry>
              <key><value>AM</value></key>
              <value>会见客户</value>
          </entry>
          <entry>
              <key><value>PM</value></key>
              <value>开会</value>
          </entry>
      </map>
  </property>
  <!-- 配置property属性 -->
  <property name="emails">
      <props>
          <prop key="john">johnson@gmail.com</prop>
          <prop key="gate">gate@gamil.com</prop>
      </props>
  </property>
  <!-- 通过util命名空间配置集合类型 -->
  <util:list id="favorites" list-class="java.util.LinkedList">
      <value>book</value>
      <value>music</value>
  </util:list>
  <util:map id="email" map-class="java.util.HashMap">
      <entry key="john" value="john@gamil.com"/>
      <entry key="gate" value="gate@gamil.com"/>
  </util:map>
  ```

  

* 简化配置方式

  ![](assets/images/di_simplified1.png)

  ![](assets/images/di_simplified2.png)

  ```xml
  <!-- 使用属性替换子元素配置bean简化配置 -->
  <bean id="carBean" class="com.smart.atrr.Car">
      <property name="brand" value="BMW"/>
      <property name="price" value="200000.0"/>
      <property name="maxSpeed" value="200"/>
  </bean>
  <bean id="boss" class="com.smart.attr.Boss">
      <property name="car" ref="carBean"/>
  </bean>
  
  <!-- 使用p命名空间配置属性进一步简化配置 -->
  <bean id="carBean" class="com.smart.attr.Car"	
        p:brand="BMW"
        p:price="200000.0"
        p:maxSpeed="200"/>
  <bean id="boss" class="com.smart.attr.Boss"
        p:car-ref="carBean"/>
  ```

  

* 自动装配

  `bean`元素提供一个`autowire`的属性，用于配置容器自动完成依赖注入，该属性值可以是下图4种之一。

  ![](assets/images/auto_wire_types.png)

  `beans`元素的`default-autowire`属性可以配置全局自动装配模式，默认情况该值设置为no，子元素的`autowire`属性可以覆盖`default-autowire`的配置。

  在实际开发中，xml的配置形式很少采用自动装配功能，而在Java注解配置方式则默认采用`byType`的自动装配策略。

  

## Bean的作用域

![](assets/images/bean_scope.png)

![](assets/images/web_request_listenter.png)

## `<bean>`元素之间的关系

### 继承关系

```xml
<bean id="abstractCar" class="com.smart.attr.Car"
      p:brand="BMW" p:price="200000.0" p:maxSpeed="200"
      abstract="true"/>
<bean id="slowCar" p:maxSpeed="150" parent="abstractCar"/>
<bean id="fastCar" p:maxSpeed="250" parent="abstractCar"/>
```

### 依赖关系

Spring配置文件的bean通过`depends-on`属性显示的指定bean前置依赖的bean，前置依赖的bean会在本bean实例化之前创建好。

```xml
<!-- sysyInit在manager实例化之前创建好 -->
<bean id="manager" class="com.smart.tagdepend.CacheManager"
      depends-on="sysyInit"/>
<bean id="sysInit" class="com.smart.tagdepend.SysInit"/>
```



### 引用关系

```xml
<property name="carId">
    <idref bean="car"/>
</property>
```

## 整合多个配置文件

```xml
<!-- 本文件名为bean2.xml -->
<!-- boo1引用的car1在bean1.xml文件中定义 -->
<!-- spring装载了本文件bean2.xml就相当于包含了bean1.xml -->
<import resourece="classpath:com/smart/impt/bean1.xml"/>
<bean id="boss1" class="com.smart.fb.Boss" p:name="john" p:car-ref="car1"/>
```

## FactoryBean

通过Java反射的方式实例化一些配置复杂的类时比较繁琐，这时可以通过编码的方式实例化，`FactoryBean` 接口就是为了达到隐藏实例化一些复杂Bean的细节，给上层应用带来便利性。

`FactoryBean` 接口在 Spring 框架中占有重要的地位，Spring 本身就提供了70多个实现类。比如， `ProxyFactoryBean` 类用于动态代理类。

FactoryBean是Spring为简化配置构造复杂的bean提供的一个接口，接口方法`getObject("car")`返回的与`getBean("car")`方法相同的bean对象，如果要获取FactoryBean对象本身，则使用`getObject("&car")`

```java
public interface FactoryBean<T>{
    T getObject() throws Exception;
    Class<T> getObjectType();
    boolean isSingleton();
}
```



## 4种配置方式

### XML文件配置

### Java注解配置

Spring提供了以下4个Java注解配置bean的信息：

* `@Componet`：通用注解；
* `@Repository`：对DAO实现类进行注解；
* `@Serivice`：对Service实现类进行注解；
* `@Controller`：对Controller实现类进行注解。

配置扫描注解定义的Ban：

```xml
<context:component-scan base-package="com.smart" resource-pattern="anno/*.class">
    <context:include-filter type="regex" expression="com\.smart\.anno.*"/>
    <context:exclude-filter type="aspectj" expression="com.smart..*Controller+"/>
</context:component-scan>
```

Java注解配置bean的方式中，提供的与依赖注入相关的注解：

```java
@Autowired(required=false) //默认按类型注入，实际多使用本注解
@Qualifier("beanName")  //按bean名称注入
@Resource("beanName")  //默认按bean名称
@Inject()  //默认按类型注入，没有required属性
@Order(value=1) //值越小，优先加载
@Scope("prototype")
@PostConstruct()
@PreDestory()
@Lazy  //延迟加载
```

Spring容器创建bean时（单例模式），先调用bean实现类的无参数构造方法，再执行`@Autowired`进行自动注入，接着执行标注了`@PostConstruct`的方法，在容器关闭时，执行标注了`@PreDesotry`方法。

### Java类配置

JavaConfig是Spring的一个子项目，旨在通过Java类的方式提供bean的定义信息，Spring 4.0基于Java类配置的核心就依赖于JavaConfig。

标注了`@Configuration`注解的Java类就可以为Spring提供bean的配置信息。标注了`@Bean`注解的方法为定义一个bean提供信息，bean类型为方法返回值，bean名称为方法名，还可以通过`@Bean("beanName")`的方法自定义bean的名称，以下为一个简单的Java类配置bean的示例：

```java
@Configuration
public class AppConfig{
    @Autowired
    private UserDao userDao;
    
    @Scope("prototype")
    @Bean
    public UserDao userDao(){
        return new UserDao();
    }
}
```

使用基于Java类的配置信息启动Spring容器：

```java
public class JavaConfigTest{
    @Test
    public void javaConfigTest(){
        ApplicationContext ctx = new AnnotationConfigApplicationContext(AppConfig.class);
        UserDao userDao = ctx.getBean(UserDao.class);
    }
}
```



### Groovy DSL（Domain Specific Languages）配置

Groovy配置bean的示例：

```groovy
//文件名：spring-context.groovy
import com.smart.anno.LogDao
import com.smart.groovy.LogonService
import com.smart.groovy.UserDao

beans{
    //声明context命名空间
    xmlns context:"http://www.springframework.org/schema/context"
    //与注解混合使用，定义注解bean扫描包路径
    context.'component-scan'('base-package':"com.smart.groovy"){
        //排除不需要扫描的路径
        'exclude-filter'('type':"aspectj",'expression':"com.smart.xml.*")
    }
    //读取app-conf.properties配置文件
    def stream;
    def config = new Properties();
    try{
        stream = new ClassPathResource('app-config.properties').inputStream
        config.load(stream);
    }finally{
        if(stream != null){
            stream.close()
        }
    }
    
    //配置无参构造bean
    logDao(LogDao){
        bean->
        bean.scope = "prototype"
        bean.initMethod = "init"
        bean.destoryMethod = "destory"
        bean.lazyInit = true
    }
    
    //根据条件配置bean，是Groovy DSL的一大特色
    if("db" = config.get("dataProvider")){
        userDao(UserDao)
    }else{
        userDao(XmlUserDao)
    }
    
    //配置有参构造函数注入bean，参数是userDao
    longonService(LogonService, userDao){
        //属性注入，引用Groovy定义的bean logDao
        logDao = ref("logDao")  
        //属性注入，引用注解定义的bean mailService
        mailService = ref("mailService")
    }
}
```

调用Groovy配置文件创建容器的示例：

```java
package com.smart.groovy;

public class LogonServiceTest{
    @Test
    public void getBean(){
        //加载Groovy定义的配置文件创建容器
        ApplicationContext ctx = new GenericGroovyApplicationContext("classpath:com/smart/groovy/spring-context.groovy");
        //加载Groovy定义的bean
        LogonService logonService = ctx.getBean(LogonService.class);
        assertNotNull(logonService);
        
        //加载注解定义的bean
        MailService mailService = ctx.getBean(MailSerivce.class);
        assertNotNull(mailService);
        
        //判断注入的是否为DdUserDao
        UserDao userDao = ctx.getBean(UserDao.class);
        assertTrue(userDao instanceof DbUserDao)
    }
}
```

### 不同配置之间的比较



![](assets/images/bean_config_methods.png)

![](assets/images/usecases.png)

## Spring容器的内部工作机制

Spring的AbstractApplicationContext是ApplicationContext的抽象实现类，该抽象类的`refresh()`方法定义了Spring容器在加载配置文件后的各项处理过程，这些处理过程清晰的刻划了Spring容器启动时所执行的各项操作。

```java
public void refresh() throws BeansException, IllegalStateException {
		synchronized (this.startupShutdownMonitor) {
			// Prepare this context for refreshing.
			prepareRefresh();
			// Tell the subclass to refresh the internal bean factory.
			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
			// Prepare the bean factory for use in this context.
			prepareBeanFactory(beanFactory);
			try {
				// Allows post-processing of the bean factory in context subclasses.
				postProcessBeanFactory(beanFactory);
				// Invoke factory processors registered as beans in the context.
				invokeBeanFactoryPostProcessors(beanFactory);
				// Register bean processors that intercept bean creation.
				registerBeanPostProcessors(beanFactory);
				// Initialize message source for this context.
				initMessageSource();
				// Initialize event multicaster for this context.
				initApplicationEventMulticaster();
				// Initialize other special beans in specific context subclasses.
				onRefresh();
				// Check for listener beans and register them.
				registerListeners();
				// Instantiate all remaining (non-lazy-init) singletons.
				finishBeanFactoryInitialization(beanFactory);
				// Last step: publish corresponding event.
				finishRefresh();
			}
			catch (BeansException ex) {
				if (logger.isWarnEnabled()) {
					logger.warn("Exception encountered during context initialization - " +
							"cancelling refresh attempt: " + ex);
				}
				// Destroy already created singletons to avoid dangling resources.
				destroyBeans();
				// Reset 'active' flag.
				cancelRefresh(ex);
				// Propagate exception to caller.
				throw ex;
			}
			finally {
				// Reset common introspection caches in Spring's core, since we
				// might not ever need metadata for singleton beans anymore...
				resetCommonCaches();
			}
		}
	}
```

![](assets/images/container_flow.png)

Spring组件按其所承担的角色可以分为两类：

* 物料组件：`Resource`, `BeanDefinition`, `PropertyEditor`以及最终的`Bean`等，它们是加工流程中被加工、被消费的组件，就像流水线上被加工的物料一样。
* 设备组件：`ResourceLoader`, `BeanDefinitionReader`, `BeanFactoryPostProcessor`, `InstantiationStrategy`及`BeanWapper`等，他么就像流水线上不同环节的加工设备，对物料组件进行加工。

`JavaBean`和`BeanInfo`，`BeanInfo`中的`getPropertyDescriptors()`方法，返回对应`JavaBean`的属性的`PropertyDescriptor`。`PropertyDescritor`的构造方法有两个参数`PropertyDescriptor(String propertyName, Class beanClass)`，其中`propertyName`对应JavaBean的参数，而`beanClass`则为JavaBean对应的class，`PropertyDescriptor`还有一个`setPropertyEditorClass(Class propertyEditorClass)`方法，用于为当前属性值指定属性编辑器，这样，一个JavaBean的属性通过一个`PropertyDescriptor`对象描述出来。

