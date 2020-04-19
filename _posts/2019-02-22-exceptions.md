# Exception

## Exception 的分类

>  Use checked exceptions for recoverable conditions and runtime exceptions for programming errors. 
>
> *Effective Java*

![Java Checked Vs Unchecked Exception - Crunchify](assets/images/Java-Checked-Vs-Unchecked-Exception-Crunchify.png)

## Checked Exception

### 一般概念

编译器在编译期对可能抛出的异常进行检查的机制，编译器要求对可能抛出 Checked Exception 必须通过 `try catch` 进行处理，或调用抛出异常异常方法的方法签名 `throws` 抛出的异常。

比如，给 `FileReader` 类的构造方法一个文件的路径，但该文件不存在，或程序没有读取权限，则会抛出 `FileNotFoundException` 异常（ 该异常是 `IOException` 的子类）。

* IOException
* SQLException
* DataAccessException
* ClassNotFoundException
* InvocationTargetException
* MalformedURLException

### 处理方式

- log it and return
- rethrow it (declare it to be thrown by the method)
- construct a new exception by passing the current one in constructor

## Unchecked Exception

编译器不会在编译期对可能抛出的异常进行检查，所有的 Unchecked Exception 都是 `RuntimeException` 的子类。

比如， 给 `FileReader` 类的构造方法传递一个空指针的文件路径，则会抛出 `NullPointerException` 。

- NullPointerException
- ArrayIndexOutOfBound
- IllegalArgumentException
- IllegalStateException
- NumberFormatException

## Error

`FileReader` 类顺利获得文件，但是由于硬件或系统的问题，不能正常读取数据，则会抛出 `IOError` 。



