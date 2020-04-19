# spring boot notes

基于spring framework 的web应用开发需要大量基于xml或注解的配置，在开发web应用需要将精力放在配置上。为了减轻配置压力，将精力集中在应用的开发上，spring team在spring framework 的基础上开发了spring boot 框架。基于spring boot的开发的web应用的原理与spring framework相同，只是将大量的配置工作由spring boot承担。

Spring Boot 的核心思想是“约定优于配置（Convention Over Configuration）”，减少应用开发过程中需要做决定的数量，仅需配置应用中与约定不符的部分。使得应用的开发、测试和部署便捷高效。

## spring boot构成组件

1. starter：将开发web应用需要的各种相关依赖整合为一个相互兼容的`spring-boot-starter-*`依赖。
2. auto configurator：根据当前的配置，自动配置web应用需要用到的各个组件。
3. spring cli：通过命令行运行基于spring boot的网络应用。
4. actuator：管理spring boot应用，提供各个应用的检测数据，图表。

## 创建spring boot项目的途径

1. [starter](start.spring.io)
2. spring tool suite
3. spring cli
4. 集成开发环境（ide）的支持



init a spring boot project:

```shell
$ spring init -dweb,data-jpa,thymeleaf,h2 --build gradle -a readinglist -g com.github.yuanqunwang readinglist
```

inspect dependencies:

```shell
$ gradle dependencies
```

```java
@SpringBootApplication
public class DemoApplication {

        public static void main(String[] args) {
                SpringApplication.run(DemoApplication.class, args);
        }

}
```

上述`DemoApplication`类的作用有两个：

1. 处于配置Spring中心的类，配置包含以下几个方面：

   由`@SpringBootApplication` 注解提供了组件扫描（Component-scanning）和Spring boot 的自动配置（auto configuration） 功能。该注解自身由以下三个注解组合而成：

   * spring的`@Configuration`，基于java类配置spring bean的注解，标注了`@Configuration`注解的类，可以通过在其方法上标注`@Bean`注解，实现java bean的定义。
   * spring的`@Componentscan`，没有任何参数的情况下，`@ComponentScan`注解会扫描当前包及其子包内，所有标注了`@Component`注解的类（其中就包含了`@Configuration`注解，因为`@Component`是其元注解），并将其加载到spring容器中。
   * spring boot的`@EnableAutoConfiguration`，实现spring boot的自动配置功能。

2. 启动（bootstrap）spring

   通过`SpringApplication.run(DemoApplication.class, args)` 方法启动spring。

## spring boot 提供的主要功能

* 自动化配置：根据类路径下的包，以及`application.properities`配置文件，自动生成相关的bean。
* 依赖整合：通过`spring-boot-starter-*`自动整合相关的、互相兼容的依赖包，而不用手动逐一添加依赖。
* 提供监控功能能。

