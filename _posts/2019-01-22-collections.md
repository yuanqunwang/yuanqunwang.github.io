# Collections

## Java集合架构

![](assets/images/java_collection_hierarchy.png)

## 常用集合

<img src="assets/images/collection_implementions.png" style="zoom:50%;" />

### 共性

#### 常用方法

```java
boolean add(E e);
boolean addAll(Collection<? extends E> c);

boolean remove(Object o);
boolean removeAll(Collection<?> c);
boolean retainAll(Collection<?> c);
void clear();

boolean contains(E e);
boolean containsAll(Collection<?> c);	//与c的交集
Iterator<E> iterator();
boolean isEmpty();
int size();

Object[] toArray();
<T> T[] toArray(T[] a);
```

* 需要用到比较集合中的元素的情况下，对应的元素应该实现相应的`equals`方法，才会产生正常的结果。

#### ForEach

除了数组可以使用`foreach`之外，从JDK 1.5开始，实现了`Iterable`就可以使用`foreach`。

#### Iterator

```java
//集合可以调用iterator方法获得Iterator对象
Iterator<E> iterator();

//Iterator对象有如下方法
boolean hasNext();
E next();
void remove();

//List除了还有listIterator方法
ListIterator<E> iterator(int n)	//n为可选参数，表示从n位置获得ListIterator
//ListIterator除了具有Iterator所有的方法之外，还有如下方法
int nextIndex();
E previous();
int previousIndex();
void set(E e)	//将上一次privious或next返回的对象替换为e
```

#### Collections工具类

#### `Comparable` 和 `Comparator`

##### `Comparable`

`Comparable` 接口的声明如下：

```java
package java.lang;
public interface Comparable<T>{
    int compareTo(T o)
}
```

`Arrays` 和 `Collections` 工具类的`sort` 方法在对元素进行默认排序时，会要求元素实现了 `Comparable` 接口，否则会抛出 `ClassCastException` 异常。Java中的`String`，`Integer` 等类都实现了该接口。

##### `Comparator`

`Comparator` 接口的声明（部分方法）如下：

```java
package java.util;
public interface Comparator<T>{
    int compare(T o1, T o2);
}
```

当需要定制排序规则时，通过实现 `Comparator` 接口，重写 `compare` 方法，制定自定义排序规则，将实现对象传给工具类的 `sort` 方法，实现自定义规则排序。通常实现 `Comparator` 接口的对象作为匿名内部类作为参数，或采用lambda表达式。`Comparator` 的使用是一种策略模式（ strategy pattern ）。

### List

#### ArrayList

ArrayList随机读写速度快，插入和删除速度慢。

常用方法

```java
int indexOf(Object o);	//返回元素位置，否则返回-1
int lastIndexOf(Object o);
E get(int index);	//返回index位置的元素
E set(int index, E element);	//将list中位置为index的元素替换为element，返回该位置之前的元素

/* 包含fromIndex，但不含toIndex
 * 返回list子集，返回的子集对应原list中的元素，对子集元素的修改会作用到原list中，反之亦然
 * 不通过返回的子集的结构变化，会导致不可预知的结果
 * 删除集合中的子集：list.subList(from, to).clear();
 */
List<E> subList<int fromIndex, int toIndex);
```

#### LinkedList

LinkedList随机读写速度慢，插入和删除速度快，特性与ArrayList相反。

### Set

#### HashSet

* 随机读写速度快，但是元素的顺序与插入顺序无关。低层使用hash作为索引。
* 元素必须重写`hashCode` 和 `equals` 方法。
* 如无其他需求，默认选择该实现。

#### TreeSet

* 元素在set中是按顺序存储的，利用红黑树实现元素的排序。
* 元素必须重写 `equals` 方法以及实现 `Comparable` 接口。

#### HashLinkedSet

* 保持读写速度的同时，还保证了元素的存储顺序与插入顺序相同。
* 元素必需重写 `equals` 和 `hashCode` 方法。

### Queue

`Queue`是一个先进先出的集合（FIFO，first in, first out）。从队列的一端读数据，另一端写数据。`Deque`（`Deque`= 'double queue'，通常发音为‘deck’）是一个支持从队列两端读写数据的集合，`Deque` 是`Queue` 的子接口。有些双端队列接口会限制队列中元素的数量，但是`Deque`接口同时支持固定元素和元素数量不限的两种形式。

4 

<img src="assets/images/deque_methods.png" style="zoom:50%;" />

<img src="assets/images/deque_vs_queue.png" style="zoom:50%;" />



#### PriorityQueue

不是根据元素进入的顺序读取，而是根据一定的优先级出队。一般而言，是队列中元素的“自然顺序”，即元素实现`Comparable` 接口的排序。也可以在队列创建时指定比较器来实现排序。

#### ArrayDeque

#### LinkedDeque

### Stack

是一个先进后出的集合（FILO，first in, last out），虽然存在`java.util.Stack` 实现类，但`java.util.LinkedList` 能提供一个更好的`Stack`， 所以，一般都使用`LinkedList`做为一个`Stack`使用。

将`LinkedList`作为一个`Stack`使用时，方法的对应关系如下：

<img src="assets/images/stack_vs_deque.png" style="zoom:50%;" />

调度场算法(shunting yard algorithm)将中缀表达式转换为后缀表达式（抽象语法树）时，处理运算符的优先级时，用到了`Stack`的先进后出的特性。

### Map

map又称为associative array，将一个对象与另一个对象建立联系。

#### HashMap

* 随机读写速度快，但是元素的顺序与插入顺序无关。低层使用hash作为索引。使用示例

  ```java
  for(Map.Entry entry : System.getenv().entrySet()){
      System.out.println(entry.getKey() + ":" + entry.getValue());
  }
  ```

* 如无其他特殊需求，默认使用该实现。

* 元素必须重写 `hashCode` 和 `equals` 方法。

#### TreeMap

* 利用红黑树实现元素的排序。
* 必须重写 `equals` 方法以及实现 `Comparable` 接口。

#### HashLinkedMap

* 保持读写速度的同时，还保证了元素的存储顺序与插入顺序相同。
* 元素必须重写 `hashCode` 和 `equals` 方法。

### Queue

FIFO( first in first out)，Java中`LinkedList`实现了`queue`接口，只需将`LinkedList`强制类型转换为`queue`即可。

```java
Queue<String> qs = new LinkedList<String>();

boolean offer(E e);	//向queue中插入一个元素
E peek();	//返回第一个元素，且不删除元素，为空则返回null
E element();	//返回第一个元素，且不删除元素，为空则抛出NoSuchElementException异常
E poll();	//返回第一个元素，并且删除该元素，为空则返回null
E remove();	//返回第一个元素，并且删除该元素，为空则抛出NoSuchElementException异常
```

`Queue`完全按照元素插入的顺序取出，有时元素进入顺序并不是最重要的，需要按某个特定需求取出，这时就用到`PriorityQueue`，在创建`PriorityQueue`对象时，指定`Comparator`确定元素取出顺序，如下所示。其余方法与`Queue`相同，但不含`element()`方法。

```java
PriorityQueue<Integer> priorityQueue = new PriorityQueue<Integer>(Collections.reverseOrder());
```

### Stack

LIFO( last in first out ) ，Java中`LinkedList`实现了`deque`接口，包含`stack`中的方法，只需将`LinkedList`强制类型转换为`stack`即可。

```java
Stack<String> ss = new LinkedList<String>();

void push(E e);		//向stack中插入一个元素
E pop();	//返回并删除stack中的第一个元素，为空则抛出NoSuchElementException异常
E peek();	//返回第一个元素，且不删除元素，为空则返回null
```

