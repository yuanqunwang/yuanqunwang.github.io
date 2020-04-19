# 动态代理

动态代理是相对于编译期或类加载期生成的代理而言的，动态代理是指在程序运行期间生成的代理。

## JDK动态代理

JDK动态代理只能为接口生成代理类。`Proxy.getProxyClass`方法动态的生成实现了接口类的字节码文件。

代理类构造方法：方法参数为`InvocationHandler`实例。

代理类如何实现接口方法：对接口方法的调用，都会分发到代理实例中，`InvocationHandler`实例`invoke`方法的调用。

代理类和目标类实现了相同的代理接口。当调用代理实例的接口方法时，该方法最终将调用`InvocationHandler`接口的`invoke(Object proxy, Method method, Object[] args)` 方法。

JDK动态代理用到的主要类：

### `Proxy`

该类主要有以下静态方法：

```java
static Class<?> getProxyClass(ClassLoader loader, Class<?>... interfaces);
static Object newProxyInstance(ClassLoader loader, Class<?>... interfaces, InvocationHandler handler);
static boolean isProxyClass(Class<?> clazz);
static InvocationHandler getInvocationHandler(Object proxy);
```

### JDK实现动态代理的步骤

```java
//代理接口
interface ProxyInterface{
  void sayHello(String msg);
}

//目标类
class TargetClass implements ProxyInterface{
  public void sayHello(String msg){
    System.out.println("From TargetClass. Hello World, " + msg);
  }
}

//MyInvocationHandler
class MyInvocationHandler implements InvocationHandler{
  private Object target;
  
  public MyInvocationHandler(Object obj){
    this.target = obj;
  }
  
  public Object invoke(Object proxy, Method method, Object[] args) throws Throwable{
    System.out.println("in MyInvocationHandler before invoke target method.");
    Object result = method.invoke(target, args);
    System.out.println("in MyInvocationHandler after invoke target method.");
  }
}

public class JDKProxyDemo{
  TargetClass target = new TargetClass();
  InvocationHandler myHandler = new MyInvocationHandler(target);
  
  //1. 代理类的获取及使用
  Class<?> proxyClass = Proxy.getProxyClass(JDKProxyDemo.class.getClassLoader(), 
                                            new Class[] { ProxyInterface.class });
  ProxyInterface proxyInstance =
    Proxy.getConstructor(new Class[]{InvocationHandler.class}).newInstance(myHandler);
  
  proxyInstance.sayHello("instance from proxy class");
  
  //2. 直接生成代理实例
  ProxyInterface proxyInstance1 =Proxy.newProxyInstance(
    JDKProxyDemo.class.getClassLoader(), 
    new Class[] { ProxyInterface.class },
    myHandler);
  
  proxyInstance1.sayHello("instance directly from Proxy.newProxyInstance()");
  
  //3. 获得代理实例的InvocationHandler实例
  InvocationHandler handler = Proxy.getInvocationHandler(proxyInstance1);
  assertEquals(myHandler, handler);
  
  //4. 判断类是否为代理类
  assertTrue(Proxy.isProxyClass(proxyClass));
}
```



## Cglib动态代理

Cglib(Code Generation Library)通过直接修改目标类的字节码文件，拦截目标类方法的调用。动态地创建目标类的子类，从而实现动态代理。

Cglib实现的动态代理不需要目标类或代理类实现代理接口。

Cglib库底层使用ASM库操作Java字节码。

* 用到cglib库的项目：
  * spring
  * hibernate
  * mockito

根据需求，可选用不同的拦截器。

```java
// 目标类
class Target{
  public void sayHello(){
    System.out.println("sayHello from Target.");
  }
}

// 拦截器
class MyMethodInterceptor implements MethodInterceptor{
  private Enhancer enhancer = new Enhancer();
  
  public Object getProxy(Class clazz){
    enhancer.setSuper(clazz);
    enhancer.setCallback(this);
    return enhancer.create();
  }
  
  public Object intercept(Object obj, Method method, Object[] args, MethodProxy methodProxy){
    System.out.println("proxy runs before Target.");
    Object result = methodProxy.invokeSuper(obj, args);
    System.out.println("proxy runs after Target.");
    return result;
  }
}

// 测试Cglib动态代理
public class CglibTest{
  @Test
  public void testDynamicProxy(){
    Target targetProxy = (Target)MyMethodInterceptor.getProxy(Target.class);
    targetProxy.sayHello();
  }
  
  @Test
  public void testDynamicProxy1(){
    Enhancer enhancer = new Enhancer();
    enhancer.setSuperclass(Tartet.class);
    enhancer.setCallback(new MyMethodInterceptor());
    Target targetProxy = (Target) enhancer.create();
    targeteProxy.sayHello();
  }
}
```

* callback种类：
  * FixedValue
  * MethodIterceptor
  * InvocationHandler
* 拦截特点
  * 方法修饰符为`final`时，不会被拦截。

### Spring框架中动态代理

1. 当目标类实现了接口，Spring默认使用JDK动态代理。
2. 当目标类未实现接口，Spring则默认使用Cglib动态代理。
3. 可以通过设置，强制使用JDK动态代理。

