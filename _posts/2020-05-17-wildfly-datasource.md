---
title: WildFly 配置数据源
tags: [WildFly, Java EE]
---

## Standalone模式配置mysql数据源

### 添加Mysql-connector模块

在`$WILDFLY_HOME/modules/system/layers/base`文件夹下创建如下目录：`com/mysql/main`。将`mysql-connector-java-8.0.20.jar`文件复制到该目录下，同时创建`module.xml`文件。

```shell
$ pwd
 /usr/local/wildfly/modules/system/layers/base/com/mysql/main
$ ls
 module.xml                      mysql-connector-java-8.0.20.jar
```

`module.xml`文件的内容如下：

```shell
<module xmlns="urn:jboss:module:1.5" name="com.mysql">
    <resources>
        <resource-root path="mysql-connector-java-8.0.20.jar" />
    </resources>
    <dependencies>
        <module name="javax.api"/>
        <module name="javax.transaction.api"/>
    </dependencies>
</module>
```

### 配置standalone.xml文件

```xml
<subsystem xmlns="urn:jboss:domain:datasources:4.0">
    <datasources>
        <datasource jta="true" jndi-name="java:/jboss/datasources/MysqlDS" pool-name="MysqlDS" enabled="true" use-ccm="true">
            <connection-url>jdbc:mysql://localhost:3306/mysqldb</connection-url>
            <driver>mysql</driver>
            <security>
                <user-name>admin</user-name>
                <password>admin-pass</password>
            </security>
            <validation>
                <valid-connection-checker class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLValidConnectionChecker"/>
                <background-validation>true</background-validation>
                <exception-sorter class-name="org.jboss.jca.adapters.jdbc.extensions.mysql.MySQLExceptionSorter"/>
            </validation>
        </datasource>
		//其他datasource
        <drivers>
            <driver name="mysql" module="com.mysql">
                <xa-datasource-class>com.mysql.jdbc.Driver</xa-datasource-class>
            </driver>
			//其他driver
        </drivers>
    </datasources>
</subsystem>
```

### 在`persistence.xml`文件中引用配置好的数据源

```xml
<persistence version="2.1"
    xmlns="http://xmlns.jcp.org/xml/ns/persistence"         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
    xsi:schemaLocation="
        http://xmlns.jcp.org/xml/ns/persistence
        http://xmlns.jcp.org/xml/ns/persistence/persistence_2_1.xsd">
    <persistence-unit name="primary">
        <jta-data-source>java:/jboss/datasources/MysqlDS</jta-data-source>
        <properties>
            <property name="hibernate.archive.autodetection" value="class" />
            <property name="hibernate.show_sql" value="true" />
            <property name="hibernate.hbm2ddl.auto" value="create" />
        </properties>
    </persistence-unit>
</persistence>
```



参考资料

* [Install and Configure MySQL JDBC Driver on JBoss Wildfly](https://medium.com/@hasnat.saeed/install-and-configure-mysql-jdbc-driver-on-jboss-wildfly-e751a3be60d3)
