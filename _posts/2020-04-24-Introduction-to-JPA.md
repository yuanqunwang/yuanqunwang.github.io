---
title: Introduction to JPA
tage: [JPA, Database, Framework]
---

## JPA 是什么

JPA（ Java Persistence API ）是一个 Java 应用程序接口规范 （ specification ），描述在Java标准版平台（ Java SE ）和 Java 企业版平台（ Java EE） 应用中 **关系型数据（ relational data ）** 的管理。参考实现（ reference implementation）为 EclipseLink。其他常见实现有 Hibernate，Toplink， Spring Data JPA 等。

持久化在此有三个层面的意思：

* API 本身，定义在 javax.persistence 包中。
* Java 持久化查询语言（JPQL，Java Persistence Query Language ）。
* 对象/关系 元数据。

## JPA 的类体系结构

![](img/jpa_class_relationships.png)

| 类                     | 作用                                                         |
| ---------------------- | ------------------------------------------------------------ |
| `Persistence`          | 提供用于获得 `EntityManagerFactory` 实例的静态方法。         |
| `EntityManagerFactory` | 是创建 `EntityManager` 实例的工厂类，可以创建和管理多个 `EntityManager` 实例。 |
| `EntityManager`        | 是一个接口，管理对象的持久化操作，同时，也是 `Query` 实例的工厂类。 |
| `EntityTransaction`    | 用于管理 `EntityManager` 的数据库事务，同时与其为一对一的关系。 |
| `Query`                | 是一个接口，用于获取满足查询条件的数据库记录。               |
| `Entity`               | 代表需要持久化的对象，对应数据库表中的一行记录。             |

## ORM

ORM（ Object Relational Mapping ）提供了对象与数据库记录之间相互转换的能力。这个转换就是通过 mapping.xml 文件指导 JPA 如何完成这个转换，此外通过 Java 注解可以实现相同的转换功能，同时，可以获得更好的维护性（`Entity` 类和注解在同一个文件中，而mapping.xml文件则有两个文件需要管理）。

## 使用实例

persistence.xml 配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>

<persistence version="2.0" xmlns="http://java.sun.com/xml/ns/persistence"
   xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
   xsi:schemaLocation="http://java.sun.com/xml/ns/persistence 
   http://java.sun.com/xml/ns/persistence/persistence_2_0.xsd">
   
   <persistence-unit name="Eclipselink_JPA" transaction-type="RESOURCE_LOCAL">
   
      <class>com.tutorialspoint.eclipselink.entity.Employee</class>

      <properties>
         <property name="javax.persistence.jdbc.url" value="jdbc:mysql://localhost:3306/jpadb"/>
         <property name="javax.persistence.jdbc.user" value="root"/>
         <property name="javax.persistence.jdbc.password" value="root"/>
         <property name="javax.persistence.jdbc.driver" value="com.mysql.jdbc.Driver"/>
         <property name="eclipselink.logging.level" value="FINE"/>
         <property name="eclipselink.ddl-generation" value="create-tables"/>
      </properties>
      
   </persistence-unit>
</persistence>
```

`Employee` 类

```java
@Entity
@Table
public class Employee {

   @Id
   @GeneratedValue(strategy = GenerationType.AUTO) 	
   
   private int eid;
   private String ename;
   private double salary;
   private String deg;
  //constructors, getters, setters...
}
```

实例：

```java
public class CreateEmployee {

   public static void main( String[ ] args ) {
   
      EntityManagerFactory emfactory = Persistence.createEntityManagerFactory( "Eclipselink_JPA" );
      
      EntityManager entitymanager = emfactory.createEntityManager( );
      entitymanager.getTransaction( ).begin( );

      Employee employee = new Employee( ); 
      employee.setEid( 1201 );
      employee.setEname( "Gopal" );
      employee.setSalary( 40000 );
      employee.setDeg( "Technical Manager" );
      
      entitymanager.persist( employee );
      entitymanager.getTransaction( ).commit( );

      entitymanager.close( );
      emfactory.close( );
   }
}
```







