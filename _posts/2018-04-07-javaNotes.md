---
title: Misc Java Facts
categories: Java
tags: [Java, Java SE, Java EE, Java Me, Java FX]
---
* A platform is the hardware or software environment in which a program runs. We've already mentioned some of the most popular platforms like Microsoft Windows, Linux, Solaris OS, and Mac OS. Most platforms can be described as a combination of the operating system and underlying hardware. The Java platform differs from most other platforms in that it's a software-only platform that runs on top of other hardware-based platforms.
* Java is being continually extended to provide language features and libraries that elegantly handle problems that are difficult in traditional programming languages, such as concurrency, database access, network programming, and distributed computing. Java allows client-side programming via the applet and with Java Web Start.
* JavaFX is a set of graphics and media packages that enables developers to design, create, test, debug, and deploy rich client applications that operate consistently across diverse platforms.
* all objects are inherited from the single root class Object
* To produce an unambiguous name for a library, the Java creators want you to use your Internet domain name in reverse since domain names are guaranteed to be unique. Since my domain name is MindView.net, my utility library of foibles would be named net.mindview.utility.foibles. After your reversed domain name, the dots are intended to represent subdirectories.
* When you say something is static, it means that particular field or method is not tied to any particular object instance of that class. So even if you’ve never created an object of that class you can call a static method or access a static field. With ordinary, non-static fields and methods, you must create an object and use that object to access the field or method, since non-static fields and methods must know the particular object they are working with.Now even if you make two StaticTest objects, there will still be only one piece of storage for StaticTest.i. Both objects will share the same i.An important use of static for methods is to allow you to call that method without creating an object. This is essential, as you will see, in defining the main( ) method that is the entry point for running an application.
* The default values are only what Java guarantees when the variable is used as a member of a class. This ensures that member variables of primitive types will always be initialized (something C++ doesn’t do), reducing a source of bugs.

## Java 平台（ Platform )

Java 技术包含 Java 编程语言和平台两个方面的内涵。Java 编程语言是指具有特定语法的面向对象的高级语言，Java 平台是供 Java 程序运行的环境。

Java 平台则包含 Java 虚拟机（ JVM ）和 API 两方面。JVM 是一个用于运行 Java 应用的程序。API 是一系列软件组件的集合，用于开发新的 Java 应用或组件。每个 Java 平台均提供了对应的 JVM 和 API，在某个平台上开发的 Java 应用，可以运行在兼容的 Java 平台上运行，比如，在 Java SE 平台上开发的 Java 应用，可以运行在 Java SE 和 Java EE 平台上。

Java 平台分为以下几种：

* Java SE ( Java Standard Edition )，提供了 Java 编程语言的核心功能。
* Java EE ( Java Enterprise Edition)，构建于 Java SE 之上，为开发大型、可扩展、稳定的网络应用提供了额外的 API 和运行环境。
* Java ME ( Java Micro Edition )，在小型设备（ 比如移动手机 ）上运行的版本，JVM 较 Java SE 版本规格小，且 API 是 Java SE 的子集。通常是 Java EE 服务的客户端。
* Java FX ( Java )，专为开发高效 GUI 程序的版本，通常是 Java EE 服务的客户端。

## The `static` keyword

1. not use `static`:The storage is allocated and methods become available until create an object using `new` of a class.
2. using `static` keyword ensured two things:
  1). have only a single piece of storage for a particular field.
  2). using class method even if no objects are created.
3. two ways to refer to a `static` variable
  1). name it via an object, eg. `object.field;`
  2). refer it directly through its class name, eg. `StaticTest.method();`
4. An important use of `static` for method is to allow you to call that method without creating an object. This is essential, as you will see, in defining the `main()` method that is the entry point for running an application.

## Analysis of java program

1. First two part:
  1)`package`
  2)`import`

2. Conventions
  1) The Package`java.lang` is implicitly included in every java code file.
  2) The first character of the class name is uppercase, while methods, field, etc. is lowercase.
  3) One file can have only one `public` class, and the corresponding java file name is same to the class name.

## Java中类加载机制

java虚拟机（jvm）通过类加载器（ClassLoader）动态加载程序执行过程中的需要用到的类，类加载器有如下3种：

1. 启动类加载器（bootstrap classloader），负责加载位于`$JAVA_HOME/lib` 文件夹下的包`rt.jar`，或`-Xbootclasspath`参数指定路径下的`rt.jar`包。
2. 扩展类加载器（extension classloader)，加载`$JAVA_HOME/lib/ext` 文件夹下的包
3. 应用类加载器（application classloader），加载除上述两者之外的类。

在启动虚拟机运行程序时，无需指定启动类和扩展类的路径，因为两者存在的位置是通过环境变量`JAVA_HOME`确定的。

加载自定义类时，

* 如果参数`-cp(--classpath)` 或环境变量`CLASSPATH`，则到对应的参数或环境变量中去寻找自定义类。如果同时设置了`-cp`和环境变量`CLASSPATH`，则以参数`-cp`指定的路径为准。
* 如果未指定该参数，则在当前目录去寻找类。设置了`-cp`或`CLASSPATH`就不会到当前目录下去加载自定义类，一般情况下就要把当前目录`.`放入`-cp`中。
* 当工程java工程时，还可以通过manifest文件指定自定义类路径。
* 出现错误"找不到或无法加载主类: org.mypackage.HelloWorld"的原因一般是被加载类所处的工程文件目录没有通过类路径参数指定。

编译后结构如下的java工程文件目录结构如下：

```shell
/home/user/myprogram/
            |
            ---> lib\
                  |
                  ---> supportLib1.jar
                  ---> supportLib2.jar
            ---> org/  
                  |
                  ---> mypackage/
                           |
                           ---> HelloWorld.class       
                           ---> SupportClass.class   
                           ---> UtilClass.class 
```

运行该程序的命令如下（当前目录为`/home/user/myprogram`）：

```shell
$ java -cp '.:~/myprogram/lib/*' org.mypackage.HelloWorld
```

其中`:`为Linux系统下多个类路径的分隔符（Windows系统的分割符为`;`）；`*` 表示导入目录下的所有`jar`包。

### `ClassNotFoundException` vs `NoClassDefFoundError`

1) Both NoClassDefFoundError and ClassNotFoundException are related to unavailability of a class at run-time.

2) Both ClassNotFoundException and NoClassDefFoundError are related to Java classpath.

* `ClassNotFoundException`: comes in java if we try to load a class at run-time using with Class.forName() or ClassLoader.loadClass() or ClassLoader.findSystemClass() method and requested class is not available in Java.
* `NoClassDefFoundError`: in this case culprit class was present during compile time and let's application to compile successfully and linked successfully but not available during run-time due to various reason.

