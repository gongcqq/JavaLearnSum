### 1.基础篇

#### 1.1 List集合和Map集合

##### 1.1.1 List集合

###### 1.1.1.1 ArrayList的扩容机制

**ArrayList底层会维护一个Object[]数组，该数组初始大小取决于使用的构造方法：**

1. `ArrayList()`：使用无参构造方法创建ArrayList，底层会创建一个长度为零的数组；
2. `ArrayList(int initialCapacity)`：使用该构造方法的话，底层会创建一个指定容量的数组；
3. `ArrayList(Collection<? extends E> c)`：使用该构造方法，会将入参中集合的大小作为底层数组的容量。

**使用`add(E e)`方法对ArrayList底层数组容量的影响：**

当我们使用`new ArrayList()`创建一个集合时，底层默认会创建一个长度为零的数组，这时如果调用`add(E e)`方法向集合中添加元素，则会对底层数组进行扩容，第一次数组会扩容到==10==，不够的话第二次会扩容到15，第三次是22，每次容量的大小大约是前一次的1.5倍，子所以说是大约，是因为底层不是使用`size * 1.5`的方式，而是使用移位运算的方式，具体源码如下：

```java
private void grow(int minCapacity) {
    int oldCapacity = elementData.length;
    //重点就是下面这一行，通过移位运算得出底层数组最新的容量大小
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**使用`addAll(E e)`方法对ArrayList底层数组容量的影响：**

如果ArrayList集合中原来是没有元素的，当使用addAll方法向ArrayList中添加元素后，就会触发底层数组的第一次扩容，即容量扩到10，而底层数组的最终容量则是在元素的实际个数和10这两者中取最大值。

如果ArrayList中原来是有元素的，当使用addAll方法向ArrayList中添加元素后，如果元素的实际个数不超过10，底层数组则不会进行扩容；如果超过10了，那么底层数组的最终容量则是在元素的实际个数和底层数组下次扩容的大小之间取最大值。比如ArrayList中原来有5个元素，当再向ArrayList中添加6个元素后，底层数组的容量会变成15，即下次扩容的容量大小；但如果是向ArrayList中添加11个元素，由于`11 + 5 > 15`，所以底层数组的最终容量会是16。

###### 1.1.1.2 Iterator的并发问题

**Fail-Fast与Fail-Safe：**

- `ArrayList`是fail-fast的典型代表，遍历该集合时如果有别的线程修改了集合的内容，会尽快失败并抛异常，异常信息类似下面这样：

  ![image-20211220155730901](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211220221452.png) 

- `CopyOnWriteArrayList`是fail-safe的典型代表，遍历的同时可以并发修改，原理是读写分离。不会抛异常，但是会牺牲一致性。比如我们在遍历CopyOnWriteArrayList集合时，向集合中再添加一个元素进行测试，如下所示：

  ![image-20211220162329136](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211220221504.png) 

  元素添加成功后，我们放过断点，控制台打印内容如下：

  ![image-20211220163150416](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211220221520.png) 

  > **说明**：通过以上控制台打印结果可知，遍历的结果和最终打印集合的结果是不一致的，确实出现了一致性问题。

###### 1.1.1.3 ArrayList和LinkedList的区别

**ArrayList：**

1. 基于数组，需要连续内存；
2. 随机访问快(根据下标访问)；
3. 尾部插入、删除性能可以，其它部分插入、删除都会需要移动数据(比如在头部插入一个元素，那么集合中所有元素都要后移一位)，因此性能较低；
4. 可以利用CPU缓存，局部性原理，提升性能。

**LinkedList：**

1. 基于双向链表，无需连续内存；
2. 随机访问慢(要沿着链表遍历)；
3. 头尾插入、删除性能高；
4. 占用内存多。

![image-20211220210215781](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211220221541.png) 

##### 1.1.2 HashMap集合

###### 1.1.2.1 基本数据结构

`jdk1.7`：数组 + 链表；

`jdk1.8`：数组 + 链表 + 红黑树。

HashMap源码中默认是通过成员变量的方式维持一个数组，源码如下：

```java
transient Node<K,V>[] table;
```

当我们使用`new HashMap()`创建一个集合对象时，底层维护的数组默认还是null，如下所示：

![image-20211221173836633](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211221233627.png) 

当我们使用`put`方法向集合中添加元素后，就会触发底层数组扩容，第一次扩容后数组容量就会变成16，如下所示：

![image-20211221174651907](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211221233638.png) 

在底层源码中，底层维护的数组第一次扩容后的初始容量16是使用的成员变量的值，源码如下所示：

```java
/**
 * The default initial capacity - MUST be a power of two.
 */
static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```

> **说明**:以上`1 << 4`通过移位运算计算出的结果就是16。而且我们使用`put`方法向集合中添加的数据也会存到底层维护的数组中，至于存入数组后的下标(也叫索引)，是通过对放入集合中的数据的key值进行了二次哈希后再与数组容量做运算后得到的，所以如果数组扩容，这个下标是会变化的。

###### 1.1.2.2 树化与退化

**树化的规则：**

* 当底层的Node数组中某个Node的链表长度超过树化阈值8时，会先尝试通过扩容数组的方式来减少链表长度(扩容后会重新计算元素下标，所以之前下标都一样导致链表很长的元素，扩容后下标可能会发生变化，这样这些元素就不会集中在一个链表上了，链表长度就会变短了)，如果数组容量已经>=64，才会进行树化。

**树化的意义：**

* 红黑树主要是用来避免DOS攻击的，以免因为链表超长而导致性能下降，树化应当是偶然情况，是保底策略；
* hash表的查找，更新的时间复杂度是O(1)，而红黑树的查找，更新的时间复杂度是O(log~2~n)，TreeNode占用空间也比普通Node的大，所以如非必要，尽量还是使用链表；
* hash值如果足够随机，则在hash表内按泊松分布，在负载因子为0.75的情况下，底层的Node数组中某个长度超过8的链表出现概率是0.00000006，树化阈值选择8就是为了让树化几率足够小。

**退化的规则：**

* 情况1：在扩容时如果拆分树，树元素个数<=6时，会退化链表。
* 情况2：remove树节点时，若root、root.left、root.right、root.left.left有一个为null，也会退化为链表。

###### 1.1.2.3 索引的计算方法

前面有提到过，添加到集合中的数据也会存到底层维护的数组中，具体存入到数组中的什么位置，也就是数据对应数组的索引(或者叫下标)是什么，其实是通过一系列计算得出的，具体计算步骤如下：

1. 首先会调用HashMap的`hash(Object key)`方法，该方法中会进行两次哈希，第一次是通过存入集合中数据的key值的`hashCode()`方法获取一个哈希值，然后再通过这个哈希值进行二次哈希(二次哈希是为了综合高位数据，让哈希分布更为均匀)，源码如下：

   ```java
   static final int hash(Object key) {
       int h;
       return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
   }
   ```

2. 最后再通过`(capacity - 1) & hash`的方式获取到索引值，其中**capacity**是数组的容量，**hash**是经过二次哈希后得到的int类型的哈希值。对应的源码如下：

   ![image-20211221223149513](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211221233701.png) 

###### 1.1.2.4 数组容量为何是2的n次幂

上文说到，底层维护的数组第一次扩容后，容量会变为16，再次扩容的话，则通过`16 << 1`这种移位运算的方式得出下次扩容后数组的容量，也就是32，再扩容就通过`32 << 1`得到数组的容量，也就是64。

使用这种运算方式得到的数组的容量都会是2的n次幂，这种方式计算索引或者扩容时重新计算索引效率都会很高。不过带来的问题就是，hash的分散性不太好，所以索引才会需要二次hash来作为补偿，从而在一定程度上解决hash分散性不好的问题。没有采用这一设计的典型例子就是Hashtable，像Hashtable这种虽然分散性更好了，但是计算索引的效率也更低了。HashMap的设计者是在综合了各种因素后，最终选择了使用2的n次幂作为数组容量。

###### 1.1.2.5 put方法与扩容

**put方法的执行流程：**

- HashMap底层是懒惰创建数组的，底层维护的数组默认是null，只有首次使用put方法添加数据时才会创建数组；

- 数组创建后，会根据存入数据的key值计算索引，从而确定数据在数组中的位置；

- 如果计算出的索引还没有被使用，就会创建Node占位并返回；

- 如果该索引已经被占用了，主要分下面两种情况：

  - 已经是TreeNode，则走红黑树的添加或更新逻辑；

  - 是普通Node，走链表的添加或更新逻辑，如果链表长度超过树化阈值，则走树化逻辑。

- 返回前检查容量是否超过阈值，一旦超过则进行扩容。

**扩容因子(或加载因子)为什么是0.75：**

在HashMap底层会维护一个默认的扩容因子，源码如下：

```java
/**
 * The load factor used when none specified in constructor.
 */
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

默认的扩容因子是0.75的主要原因如下：

1. 在空间占用与查询时间之间取得较好的权衡；
2. 大于这个值，空间节省了，但链表就会比较长影响性能；
3. 小于这个值，冲突减少了，但扩容就会更频繁，空间占用也更多。

> **说明**：举例说明一下扩容因子的用法。比如数组的容量是16，扩容因子是0.75，两者相乘的结果就是12，那么当向数组中添加的元素个数超过12个时，就会触发数组扩容，即数组容量扩到32。

**jdk1.7和jdk1.8在插入数据以及扩容上的区别：**

- 链表插入节点时，jdk1.7是头插法，jdk1.8是尾插法；
- jdk1.7是大于等于阈值且没有空位时才扩容，而jdk1.8则是大于阈值就扩容；
- jdk1.8在扩容计算Node索引时，会进行优化。

###### 1.1.2.6 并发问题

如果是在多线程场景下操作HashMap，可能会出现以下两个问题：

- 在jdk1.7版本中，可能会出现扩容死链的问题；
- 在jdk1.7以及jdk1.8版本中，均可能出现并发丢数据的情况。

> **说明**：如果在并发场景下，多个线程同时向HashMap底层数组的同一个索引处添加元素，由于HashMap不是线程同步的，所以可能出现某一个线程添加的数据被覆盖从而导致数据丢失的情况。

###### 1.1.2.7 关于key的设计

1. HashMap的key可以为null，但是Map的其他实现则不然，比如TreeMap的key值就不能为null；
2. 如果key值是一个对象，那么则必须实现hashCode()和equals()方法，并且key的内容是不可变的。

#### 1.2 单例设计模式













### 2.Mysql篇





### 3.Redis篇





### 4.Linux篇





### 5.线程并发篇





### 6.JVM篇





### 7.Spring篇





### 8.微服务篇





### 9.网络协议篇





### 10.算法篇





### 11.人事篇

































