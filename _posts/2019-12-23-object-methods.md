`System.arrayCopy(Object src, int srcPos, Object dest, int destPost, int length)`，对于数组元素为基本数据类型时，做deep copy，数组元素为引用类型时，做shallow copy。

## `Object` 对象方法

### `clone`方法

* `clone`方法在`Object`类中的访问修饰符为`protected`。要使用`clone`方法，一般是实现`Cloneable`接口，如果没有实现该接口，则调用`clone`方法时，会抛出`CloneNotSupportException`。

* `Object#clone()`方法，对基本数据类型和不可变的引用类型（如`String`, `Intger`等），做deep copy。对引用类型做shallow copy。如果要实现deep copy，则需要对基本数据类型和不可变引用类型调用`super.clone()`方法，对引用类型则创建相应的对象，并赋值。

### `hashCode`方法

* 计算当前对象的hash值，该值为`int`类型。
* 如果两个非`null`对象`obj1`和`obj2`有以下关系：`obj1.equals(obj2) == true;`, 则这两个对象的hash值相等。
* 两个非`null`对象`obj1`和`obj2`有以下关系：`obj1.equals(obj2) == false`，两个对象的hash值可以相等，但是hash值不相等，对底层数据结构是hashtable的集合类型，可以均匀的分布到不同的散列桶中。

### `equals`方法

* 对比某个对象与当前对象是否相等。

* 默认实现是对比两个引用对象的地址是否相等，`return object == this;`。
* 使用hashtable实现的集合对象时，必须通过对比值相等重写该方法。同时也重写`hashCode`方法。

### `getClass`方法

* 返回当前对象的`Class`对象。

## 序列化

* 只有实现了`Serializable`接口的类才能序列化和反序列化。
* 实现序列化的类要有一个`private final static long serialVersionUID`属性，用于反序列化时，对比接受方对比已经加载的类和反序列化的类是否相同，如果相同则可以成功反序列化该类，否则会抛出`InvalidClassException`。