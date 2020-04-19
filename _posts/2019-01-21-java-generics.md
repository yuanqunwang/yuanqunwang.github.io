---
title: Java Generics Notes
categories: Java
tags: [Java, Generics, Functional, Stream]
---
# Java Generics

Parametric polymorphism是函数式语言的特性，Java以parameterized classes 和 polymorphic methods的形式实现了Parametric polymorphism（genericity or generics）。

## 使用范型的目的

Java范型是JDK 1.5开始推出的一个新特性，主要用于限制集合类中元素的类型，确保集合中类型正确；此外，还可以增加代码的可维护性和健壮性。

在引入范型之前，集合的使用：

```java
List myIntList = new LinkedList();
myIntList.add(new Integer(10));
//强制类型转换，开发人员认为正确的转换，但实际并不一定正确
Integer n = (Integer)myIntList.itertor().next()；
```

使用范型的代码：

```java
List<Integer> myIntList = new LinkedList<Integer>();
myIntList.add(new Integer(10));
Integer n = myIntList.itertor().next();	//没有Integer的强制类型转换
```

使用范型的代码在变量的声明时，通过`<Integer>`表明了一个元素类型是`Integer`的集合，与之前的代码不同的是，`myIntList`是一个任意元素类型的集合。在取出集合中的元素时，也省去了繁琐的强制类型转换。范型的优势在于，与传统方法相比，编译器在编译期会检查集合类型，确保集合中元素类型的正确性；同时，省去了繁琐的强制类型转换（且该转换并不一定安全）。



## 范型类型

### 接口/类

以下代码片段是从`java.util`包中选取出来的。

```java
//Itertor接口
public interface Itertor<E>{
    E next();
    boolean hasNext();
}

//List接口
public interface List<E>{
    public void add(E e);
    public Intertor<E> itertor();
}

```

与传统的接口相比，在接口声明处有一个以`<>`包围起来的**形式类型参数**`E`，在接口或类的声明内部，类型参数`E`可以像正常类型一样使用。

在调用范型类/接口时，形式类型参数（formal type parameter）都会被实际类型参数（actual type parameter），比如`List<Integer>`中，`E`出现的位置都会替换为`Integer`。但是这个替换又与传统的C++的模板不同，C++中的模板会根据类型参数扩展出多份类，而Java中范型对所有的调用只编译一次，也就是只有一份`class`字节码文件。Java中的范型可以与方法类比，方法只有一份，方法的形参相当于Java范型中的形式类型参数。

范型的形式类型参数一般选取单个大写字母，用于与其它类型作区分，集合中经常使用`E`代表Element，`T`代表Type，表中使用`K`, `V`，分别代表Key和Value。此外使用常用字母相邻的字母，比如`S`等。

### 范型方法

使用范型的方法可以使同一个方法作用于不同的数据类型，增加代码的复用性。范型方法用于在范型类型参数依赖于方法的入参类型，如果范型类型参数不依赖于方法的入参类型，则可以使用通配符。

```java
//将数组中的元素拷贝到集合中，集合中的元素类型是数组元素类型的父类。
public <T> void copyFromArrayToCollection(T[] src, Collection<T> dst){
    for(T s : src){
        dst.add(s);
    }
}
```

#### 类型推导（type inference）

## 范型和子类型（Subtyping）

* 集合元素间有父子类型的关系，但元素通过范型构造出的集合之间并没有父子类型的关系，这种变化称为*不变性*。比如：

  ```java
  List<Object> lo = new LinkedList<Object>;
  List<Integer> li = new LinkedList<Integer>;
  ```

  虽然`Integer`是`Object`的子类，但是`List<Integer>`并不是`List<Object>`的子类。该结论的一个简单推理过程：如果`List<Integer>`是`List<Object>`的子类，那么`List<String>`也是`List<Object>`的子类，这时就可以向`List<Object> lo`中加入一个`String`类型的元素，结果`lo`集合中同时有`Integer`和`String`类型的元素，这显然不能通过编译器的类型检查。

## 通配符（Wildcard）和约束（Bounds）

- `Object`是Java中所有其它类的父类，但是`List<Object>`并不是`List`集合的父类，为了表达接受多种元素是多种类型的`List`集合，通过在形式类型参数的位置处用`?`表示未知类型，`List<?>`用于接受多种类型的范型，这种类型称为wildcard type。
- ray type：没有指定范型的类型参数，作用与wildcard type相似，区别在于编译器不会对raw type进行严格的类型检查，用于支持范型代码调用JDK 1.5以下的不支持范型的代码。
- wildcard type的限制：因为`List<?>`表示元素是未知类型的集合，所以不能向该引用中增加元素，即调用该引用的`add(E e)`方法（`add(null)`例外），但可以正常调用`get()`方法。
- 通配符`?`用于表达任意类型，有时需要进一步限制元素类型，约束可以表达该需求。约束分上边界和下边界。

### `extends`

`extends`关键字表达上边界，用法可以用一下例子表示：

```java
public abstract Shape{
    public abstract void draw(Canvas c);
}

public class Circle extends Shape{
    public void draw(Canvas c){
        //drawing a circle on canvas
    }
}

public class Rectangle extends Shape{
    public void draw(Canvas c){
        //drawing a rectangle on canvas
    }
}

public class Canvas{
    public void draw(Shape s){
        s.draw(this);
    }
    
    //通过extends表达类型的上边界，接受元素类型是Shape子类的List集合
    //public void drawAll(List<Shape> ls)，只能接受元素类型为Shape的List集合
    //public void drawAll(List<?> ls)，不能确保集合元素有draw方法
    public void drawAll(List<? extends Shape> ls){
        for(Shape s : ls){
            s.draw(this);
        }
    }
}
```

以上定义中，`List<Circle>`， `List<Rectangle>`等其它元素是`Shape`子类的List集合均可作为实参传入`drawAll(List<? extends Shape>)`方法中。

### `super`

* 将元素类型为`String`的集合写入到`Object`

  ```java
  Collection<String> cs;
  Sink<Object> s;
  writeAll(cs, s);
  
  //public static <T> void writeAll(Collection<T> coll, Sink<T> s);//不能正常调用
  //public static <T> void writeAll(Collection<? extends T> coll, Sink<T> s);//T为Object
  public static <T> void writeAll(Collection<T> coll, sink<? super T> s);	//正常调用
  ```

* 比较器`Comparator`

  `TreeSet`的构造方法需要一个比较器，用于创建树时比较元素的大小。

  ```java
  public interface Comparator<T>{
      int compare(T fst, T snd);
  }
  
  TreeSet(Comparator<? super E> c);
  ```

* `Comparable`接口

  `Collections`工具类中的`max`方法的签名演进：

```java
public static <T extends Comparable<T>> T max(Collection<T> c);
public static <T extends Comparable<? super T>> T max(Collection<T> c);
public static <T extends Object & Comparable<? super T>> T max(Collection<? extends T> c);
```

### wildcard capture

## 擦除（erasure）

范型编译之后，会将形式类型参数擦除，得到一份`class`文件，即Java的范型是编译期才存在，在运行期不存在。运行期没有范型的形式类型参数信息。

```java
List<Integer> li = new ArrayList<Integer>();
List<String> ls = new ArrayList<String>();
System.out.println(li.getClass() == ls.getClass());	//true
```

### 擦除的影响

* 范型中不能使用形式类型参数`T`等创建对象、数组，不能使用`T`进行强制类型转换，调用`instanceof`。

  ```java
  class Foo<T>{
      T x;
      T[] arr;
      public Foo(){
          //x = new T();	错误
          //arr = new T[];	错误
      }
  }
  ```

* 静态变量，方法，静态块对某一范型只有一份，所有该范型的实例都共用。静态变量、静态方法以及静态块中不能使用范型的形式类型参数。

* 异常中不能使用范型。（？）

### 解决擦除的影响

* 使用`Class`类

  ```java
  class Foo<T>{
      Class<T> kind;
      T x;
      T[] arr;
      //创建对象
      public Foo(Class<T> clazz){
          this.kind = clazz;
          this.x = clazz.newInstance();
      }
      //创建对象数组
          public Foo(Class<T> clazz, int size){
          this.x = clazz.newInstance();
          this.arr = (T[])Array.newInstance(clazz, size);
      }
      
      public boolean isInstance(Object obj){
          return kind.isInstance(obj);
      }
  }
  
  public class FooTest{
      public static void main(String[] args){
          Foo<String> fs = new Foo<String>(String.class);
          Foo<String>[] fsa = new Foo<String>[](String.class, 10);
          System.out.println(fs.isInstance(new String("foo")));	//true
      }
  }
  ```

通过向构造方法中传递`String.class`参数，调用`newInstance()`即可以创建类的对象和数组（要求类有默认构造方法，对于数字类型的包装类不适用）。使用`isInstance()`方法，可以替代`instanceof`。

## 范型与legacy代码的共存

使用raw type沟通范型和legacy代码。

### 范型代码调用legacy代码

### legacy代码调用范型代码

## Variance of Generics Type

1. If a generic interface has only methods that return objects of type *T*, but don’t consume objects of type *T*, then assignment from a variable of *Type<B>* to a variable of *Type<A>* can make sense. This is called **covariance**. Examples are: I*terable<T>, Iterator<T>, Supplier<T>![inheritance](https://schneide.files.wordpress.com/2015/05/inheritance.png?w=130)*
2. If a generic interface has only methods that consume objects of type *T*, but don’t return objects of type *T*, then assignment from a variable of *Type<A>* to a variable of *Type<B>* can make sense. This is called **contravariance**. Examples are: *Comparable<T>, Consumer<T>*
3. If a generic interface has both methods that return and methods that consume objects of type *T* then it should be **invariant**. Examples are: *List<T>, Set<T>*

## 常见问题及解决方法

* a common problem when dealing with pre-Java 5 APIs. (`dom4j`)

  > "The expression of type `List` needs unchecked conversion to conform to `List<Element>`"

```java
public static <T> List<T> castList(Class<? extends T> clazz, Collection<?> c) {
    List<T> r = new ArrayList<T>(c.size());
    for(Object o: c)
      r.add(clazz.cast(o));
    return r;
}
```



## 参考资料

 [What are covariance and contravariance?](https://www.stephanboyer.com/post/132/what-are-covariance-and-contravariance)

[Generics in the Java Programming Language - Oracle](https://www.oracle.com/technetwork/java/javase/generics-tutorial-159168.pdf)

[JavaGenericsFAQ](http://www.angelikalanger.com/GenericsFAQ/JavaGenericsFAQ.html)

[Adding Wildcards to the Java Programming Language](http://www.jot.fm/issues/issue_2004_12/article5/)

[Declaration-site and use-site variance explained](https://schneide.blog/2015/05/11/declaration-site-and-use-site-variance-explained/)
