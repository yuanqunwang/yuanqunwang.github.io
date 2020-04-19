# Spring Transaction

## 数据库事务

* 数据库事务（简称事务）：是数据库管理系统执行的一个逻辑单元，是一个有限的数据库操作序列构成。当数据库事务提交给数据库管理系统执行时，则数据库管理系统需要确保事务中的所有操作都成功执行且其结果永久的保存在数据库中，如果事务中有操作没有成功完成，则事务中的所有操作都需要回滚，回到事务执行前的状态。同时，该事务对数据库中的其他事务的执行无影响，所有的事务都好像在独立的运行。
* 数据库事务的特性(ACID)：
  * 原子性(Atomicity)：事务作为一个整体被执行，包含在其中的对数据库的操作要么全部被执行，要么都不执行。
  * 一致性(Consistency)：事务应确保数据库从一个一致状态转变到另一个一致状态，一致状态的含义是数据库中的数据应满足完整性约束。
  * 隔离性(isolation)：多个并发执行的事务，相互独立，互不影响。
  * 持久性(durability)：已被提交的事务对数据库的修改被永久的保存在数据库中。

## 事务并发问题

* 脏读 Dirty read(write-read confict)：事务T1读取事务T2未提交的更新，T2可能撤销或更改更新。如果T1据此对数据库进行更新，就会导致数据库的不一致性产生。

* 不可重复读 unrepeatable read(read-write confict)：事务T1在事务T2提交更新前后读取某数据A，得到不一致的结果。

* 幻读(phantom read)：事务T1查询满足条件的数据记录，事务T2向数据库中增减或删除满足事务T1查询条件的记录，导致事务T1在事务T2前后的查询得到的结果不一致。幻读与不可重复读的区别在于前者为新增或删除记录，后者为更新记录。
* 更新丢失 lost update

## Spring事务属性

Spring的事务属性是通过`TransactionDefinition` 接口定义的一些常量表示，事务的属性包含以下方面：

* 事务传播属性
* 事务隔离级别
* 事务只读属性
* 事务超时
* 事务的回滚规则

### 事务隔离级别

隔离级别表示若干个并发事务之间的隔离程度，`TransactionDefinition` 接口中定义了表示隔离级别的5个常量：

* `TransactionDefinition.ISOLATION_DEFAULT`:使用低层数据库的默认隔离级别，一般为`TransactionDefinition.ISOLATION_COMMITED_READ`。
* `TransactionDefinition.ISOLATION_UNCOMMITED_READ`: 可以读取其他事务的未提交更改，是最低级别的隔离程度，不可防止脏读、不可重复读、幻读。
* `TransactionDefinition.ISOLATION_COMMITED_READ`：只能读取其他事务的提交更改，可以防止脏读，但是不可以防止不可重复读、幻读。
* `TransactionDefinition.ISOLATION_REPEATABLE_READ`: 同一个事务中多次的查询结果相同，即使查询之间有新的满足查询条件的数据增加。可防止脏读、不可重复读。
* `TransactionDefinition.ISOLATION_SERIALIZABLE_READ`：所有事务依次逐个执行，互不干扰，不会产生事务的并发问题。这是最高的隔离级别，但是会对程序的性能产生大影响，一般不使用。

隔离级别与事务读取现象

| Isolation level  | dirty read  | lost update | unrepeatable read | phantom read |
| ---------------- | ----------- | ----------- | ----------------- | ------------ |
| Read Uncommited  | may occur   | may occur   | may occur         | may occur    |
| Read Commited    | don't occur | may occur   | may occu          | may occur    |
| Repeatable read  | don't occur | don't occur | don't occur       | may occur    |
| Serilizable read | don't occur | don't occur | don't occur       | don't occur  |

### 事务传播行为

#### 支持当前事务

* `PROPAGATION_REQUIRED`：如果当前存在事务，则加入当前事务，如果没有，则新建事务。是最常见的选择。
* `PROPAGATION_SUPPORT`：如果当前存在事务，则加入事务，否则，以非事务状态运行。
* `PROPAGATION_MANDATORY`： 如果当前存在事务，则加入事务，否则，抛出异常。

#### 不支持当前事务

* `PROPAGATION_NEW`：新建事务，如果当前存在事务，则挂起当前事务。
* `PROPAGATION_NOT_SUPPORTED`：以非事务方式执行，如果当前存在事务，则挂起当前事务。
* `PROPAGATION_NEVER`：以非事务方式执行，如果当前存在事务，则抛出异常。
* `PROPAGATION_NESTED`：如果当前存在事务，则嵌套在当前事务中执行，否则，创建新事务。

### 事务的只读属性

事务的只读属性是指，对事务性资源进行只读操作或是读写操作。如果确定只对事务性资源进行只读操作，可以将事务标志为只读，以提高事务处理的性能。在`TransactionDefinition`中以`boolean`类型来表示事务是否只读。

### 事务超时

是指一个事务所允许执行的最长时间，如果超过该时间限制但是事务还没有完成，则自动回滚事务。在`TransactinoDefinition`中以`int`的值来表示超时时间，单位是秒。

### 事务回滚规则

通常情况下，如果在事务中抛出了未检查异常（继承自`RuntimeException`的异常），则默认回滚事务。如果没有抛出任何异常，或者抛出了已检查异常，则仍然提交事务。这是大多数开发者希望的处理方式，也是EJB中的默认处理方式。但是，我们也可以根据需要人为控制事务在抛出某些未检查异常时仍然提交事务，或者在抛出某些已检查异常时回滚事务。

## Spring中事务管理API

Spring框架，涉及到事务管理的API大约有100多个，其中最重要的有三个：

* `TransactionDefinition`
* `PlatformTransactionManager`
* `TransactionStatus`

事务管理：按照给定的事务规则来执行提交或者回滚操作。“给定的事务规则”就是用`TransactionDefinition`表示，`PlatformTransactionManager` 则执行具体的事务操作，`TransactionStatus` 则表示一个运行着的事务的状态。

如果把`TransactionDefinition` 接口比喻为程序，那么`TransactionStatus` 则为进程。

### `TransactionDefinition`接口

定义Spring中事务管理的常量，如事务的传播行为，隔离级别等。`TransactionDefinition`接口中定义了获取事务属性的方法，该接口的默认实现类是`DefaultTransactionDefinition`， 可以满足大多数需求。

### `TransactionStatus` 接口

`PlatformTransactionManager.getTransaction(...)` 方法返回一个`TransactionStatus` 对象，该对象可能代表一个新的或已经存在的事务。

该接口提供了一个简单的控制事务执行和查询事务状态的方法。

接口中定义的主要方法：

```java
public interface TransactionStatus {
  boolean isNewTransaction();
  void setRollbackOnly();
  boolean isRollbakOnly();
}
```

### `PlatformTransactionManager` 接口

该接口中定义的主要方法：

```java
public interface PlatformTransactionManager {
  TransactionStatus getTransaction(TransationDefinition definition)
    throws TransactionException;
  
  void commit(TransactionStatus status) throws TransactionException;
  
  void rollback(TransactionStatus status) throws TransactionException;
}
```

根据低层所使用的不同的持久化API或框架，`PlatformTransactionManager` 的主要实现类有：

* `DataSourceTransactionManager` : 适用于使用 JDBC 或 Mybatis 进行数据持久化操作
* `HibernateTransactionManager`：使用 Hibernate 进行数据持久化操作
* `JpaTransactionManager`: 使用 JPA 进行数据持久化操作
* 另外还有 `JtaTransactionManager`, `JdoTansactionManager`, `JmsTransactionManager` 等。



## Spring使用事务的方式

### 编程式事务管理

在代码中显示调用 `beginTransaction` , `commit`, `rollback` 等事务管理方法，这就是编程式事务管理。通过Spring提供的事务管理API，可以在代码中灵活的控制事务的执行，在底层，Spring仍然将事务操作委托给持久化框架来执行。

#### 基于低层API的编程式事务管理

编程式事务管理代码：

```java
public class BankServiceImpl implements BankService {
  private BankDao bankDao;
  private TransactionDefinition txDefinition;
  private PlatformTransactionManager txManager;
  
  public boolean transfer (long fromId, long toId, double amount) {
    //调用该方法便启动了一个事务
    TransactionStatus txStatus = txManager.getTransaction(txDefintion);
    boolean result = false;
    try {
      result = bankDao.transfer(fromId, toId, amount);
      txManager.commit(txStatus);
    } catch (Exception e) {
      result = false;
      txManager.rollback(txStatus);
      System.out.println("Transfer Error.");
    }
    return result;
  }
}
```

配置文件：

```xml
<bean id="bankService" class="footmark.spring.core.tx.programmatic.origin.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="txManager" ref="transactionManager"/>
    <property name="txDefinition">
        <bean class="org.springframework.transaction.support.DefaultTransactionDefinition">
            <property name="propagationBehaviorName" value="PROPAGATION_REQUIRED"/>
        </bean>
    </property>
</bean>
```

观察以上通过底层API进行事务管理大代码，可以发现业务逻辑代码与事务管理代码交织在一起，破坏了业务代码的逻辑性，并且每个业务方法都必须包含类似的启动事务、提交和回滚事务的代码。Spring通过使用在数据访问层常用的**模板回调模式**方法，简化事务管理。

#### 基于`TransactionTemplate`的编程式事务管理

```java
public class BankServiceImpl implements BankService {
  private BankDao bankDao;
  private TransactionTemplate txTemplate;
  
  public boolean transfer(final Long fromId, final Long toId, final Double amout) {
    return (Boolean) txTemplate.execute ( new TransactionCallback() {
      public Object doInTransaction(TransactionStatus txStatus) {
        Object result;
        try {
          //设置事务属性
          txTemplate.setIsolationLevel(TransactionDefinition.ISOLATION_READ_UNCOMMITED);
          txTemplate.setTimeout(30);  //30seconds
          result = bankDao.transfer(fromId, toId, amout);
        } catch(Exception e) {
          txStatus.setRollbackOnly();
          result = false;
          System.out.println("Transfer Error.");
        }
        return reslut;
      }
    }
    );
  }
}
```

配置文件如下：

```xml
<bean id="bankService" class="footmark.spring.core.tx.programmatic.template.BankServiceImpl">
    <property name="bankDao" ref="bankDao"/>
    <property name="transactionTemplate" ref="transactionTemplate"/>
</bean>
```

### XML配置声明式事务管理

#### 基于 `TransactionIntercptor` 和 `ProxyFactoryBean`

```xml
<beans...>
......
    <bean id="transactionInterceptor" class="org.springframework.transaction.interceptor.TransactionInterceptor">
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
    <bean id="bankServiceTarget" class="footmark.spring.core.tx.declare.origin.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>
    <bean id="bankService" class="org.springframework.aop.framework.ProxyFactoryBean">
        <property name="target" ref="bankServiceTarget"/>
        <property name="interceptorNames">
            <list>
                <idref bean="transactionInterceptor"/>
            </list>
        </property>
    </bean>
......
</beans>
```

#### 基于`TransactionProxyFactoryBean`

`TransactionProxyFactoryBean`类，用于将`TransactionInterceptor` 和 `ProxyFactoryBean` 合二为一，简化配置。

```xml
<beans......>
......
    <bean id="bankServiceTarget" class="footmark.spring.core.tx.declare.classic.BankServiceImpl">
        <property name="bankDao" ref="bankDao"/>
    </bean>
    <bean id="bankService" class="org.springframework.transaction.interceptor.TransactionProxyFactoryBean">
        <property name="target" ref="bankServiceTarget"/>
        <property name="transactionManager" ref="transactionManager"/>
        <property name="transactionAttributes">
            <props>
                <prop key="transfer">PROPAGATION_REQUIRED</prop>
            </props>
        </property>
    </bean>
......
</beans>
```

#### `tx` 和 `aop` 命名空间的声明式事务管理

```xml
<beans......>
......
    <tx:advice id="bankAdvice" transaction-manager="transactionManager">
        <tx:attributes>
            <tx:method name="transfer" propagation="REQUIRED"/>
        </tx:attributes>
    </tx:advice>

    <aop:config>
        <aop:pointcut id="bankPointcut" expression="execution(* *.transfer(..))"/>
        <aop:advisor advice-ref="bankAdvice" pointcut-ref="bankPointcut"/>
    </aop:config>
......
</beans>
```

### 注解配置声明式事务管理

在需要进行事务管理的接口、接口方法、类、类方法标注`@Transactional`注解，再在配置文件中添加如下配置，即可完成。

```xml
<beans>
  <tx:annotation-driven transaction-manager="txManager" proxy-target-class="true" order="true"/>
</beans>
```

上述配置中，如果事务管理器的名称为`transactionManage`时，可不配置。`proxy-target-class` 为true时，使用cglib生成动态代理，为false时，使用JDK接口的方式生成动态代理。`order`可控制织入业务类中切面的顺序。

* `@Transactional` 注解的默认事务属性相当于以下配置值
  * 事务传播行为：`@Transactional(propagation = Propagation.PROPAGATION_REQUIRED)`
  * 隔离级别：`@Transactional(isolation = Isolation.ISOLATION_DEFAULT)`
  * 超时：底层事务系统的默认值
  * 只读属性：读写事务，`@Transactional(readOnly = false)`
  * 回滚设置：任何运行时异常引发回滚，检查异常不会引发回滚
* `@Transactional` 注解使用实践
  * Spring 建议在业务实现类上使用该注解
  * 如果在接口或接口方法使用该注解时，只有在使用基于接口的代理才会生效。当注解驱动设置以下属性 `proxy-target-class="true"` 时，代理类将通过继承子类的方式生成，接口或接口方法标注的事务注解不会继承到生成的代理子类中，所以会导致业务类不会添加事务增强。
* 方法级注解会覆盖类级注解



