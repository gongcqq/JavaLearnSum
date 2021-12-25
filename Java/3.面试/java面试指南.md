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

##### 1.2.1 饿汉式

###### 1.2.1.1 普通饿汉式

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 17:11
 */
public class Singleton {

    private static final Singleton INSTANCE = new Singleton();

    private Singleton() {
        //这里的判断是为了防止反射破坏单例
        if (INSTANCE != null) {
            throw new RuntimeException("单例对象不能重复创建");
        }
        System.out.println("调用无参构造方法...");
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }

    //如果我们实现了Serializable接口的话，增加以下方法可以防止反序列化破坏单例，方法名是固定的
    /*public Object readResolve() {
        return INSTANCE;
    }*/
}
```

以上代码中，我们在无参构造方法中增加了一个判断，这个判断是为了防止有人通过反射的方式创建实例来破坏单例，如果我们不加这个判断，那么就可以通用反射创建出一个新的实例，演示代码如下：

```java
public class TestMain {
    public static void main(String[] args) throws Exception {
        Singleton instance = Singleton.getInstance();
        System.out.println("常规创建实例：" + instance);

        Constructor<Singleton> constructor = Singleton.class.getDeclaredConstructor();
        constructor.setAccessible(true);
        System.out.println("反射创建实例：" + constructor.newInstance());
    }
}
```

控制台打印内容如下所示：

![image-20211222210942871](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211224210606.png) 

上面的饿汉式单例模式如果实现了Serializable接口，还可以通过反序列化方式来破坏单例，增加`readResolve()`方法就是解决这一问题的。如果我们实现了Serializable接口并且没有增加readResolve()方法的话，演示效果如下：

```java
public class TestMain {
    public static void main(String[] args) throws Exception {
        Singleton instance = Singleton.getInstance();
        System.out.println("常规创建实例：" + instance);

        ByteArrayOutputStream bos = new ByteArrayOutputStream();
        ObjectOutputStream oos = new ObjectOutputStream(bos);
        oos.writeObject(instance);
        ByteArrayInputStream stream = new ByteArrayInputStream(bos.toByteArray());
        ObjectInputStream ois = new ObjectInputStream(stream);
        System.out.println("反序列化创建实例：" + ois.readObject());
    }
}
```

控制台打印内容如下所示：

![image-20211222214109467](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211224210619.png) 

然后我们在Singleton类中加入以下方法后再进行测试：

```java
public Object readResolve() {
    return INSTANCE;
}
```

控制台打印内容则如下所示：

![image-20211222214414847](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211224210627.png) 

###### 1.2.1.2 枚举饿汉式

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 17:11
 */
public enum Singleton {

    INSTANCE;

    private Singleton() {
        System.out.println("调用无参构造方法...");
    }

    @Override
    public String toString() {
        return getClass().getName() + "@" + Integer.toHexString(hashCode());
    }

    public static Singleton getInstance() {
        return INSTANCE;
    }
}
```

> **说明**：枚举饿汉式单例模式天然可以防止反射、反序列化方式破坏单例。

##### 1.2.2 懒汉式

###### 1.2.2.1 普通懒汉式

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 17:11
 */
public class Singleton {
    private Singleton() {
        System.out.println("调用无参构造方法...");
    }

    private static Singleton INSTANCE = null;

    //加锁是为了防止多线程场景下的并发问题
    public static synchronized Singleton getInstance() {
        if (INSTANCE == null) {
            INSTANCE = new Singleton();
        }
        return INSTANCE;
    }
}
```

> **说明**：其实只有首次创建单例对象时才需要使用锁进行同步，但该代码实际上是每次调用都会进行同步，而下面的双检锁懒汉式就是对这一问题的改进。

###### 1.2.2.2 双检锁懒汉式

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 17:11
 */
public class Singleton {
    private Singleton() {
        System.out.println("调用无参构造方法...");
    }

    //这里必须使用volatile进行修饰
    private static volatile Singleton INSTANCE = null;

    public static Singleton getInstance() {
        if (INSTANCE == null) {
            synchronized (Singleton.class) {
                if (INSTANCE == null) {
                    INSTANCE = new Singleton();
                }
            }
        }
        return INSTANCE;
    }
}
```

> **注意**：使用双检锁懒汉式时，必须使用==volatile==进行修饰。因为`INSTANCE = new Singleton4()`创建对象的操作不是原子的，主要分成三步：创建对象、调用构造方法、给静态变量赋值。其中后两步可能被指令重排序优化，从而变成先给静态变量赋值、再调用构造。多线程场景下，可能出现线程1执行了赋值，但还没来得及调用构造方法，然后线程2这时就执行到getInstance()方法中的第一个`INSTANCE == null`了。由于线程1已经给INSTANCE变量赋值了，所以线程2就会直接return，但是线程1还没调用构造方法，假设构造方法中是给一些别的成员变量赋值，那么就会出现线程2返回的单例对象中的一些成员变量没值的情况，而使用volatile就是为了避免指令重排序带来的这个问题。

###### 1.2.2.3 内部类懒汉式

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 17:11
 */
public class Singleton {
    private Singleton() {
        System.out.println("调用无参构造方法...");
    }

    //下面这个静态变量中的实例对象，只有在用到这个内部类的时候才会去创建，所以也是懒汉式的
    private static class Holder {
        static Singleton INSTANCE = new Singleton();
    }

    public static Singleton getInstance() {
        return Holder.INSTANCE;
    }
}
```

> **说明**：给静态变量赋值的操作最终会被放到静态代码块中执行，java虚拟机会保证静态代码块中的线程安全问题。

##### 1.2.3 jdk源码中单例的体现

###### 1.2.3.1 饿汉式单例的体现

在jdk的`java.lang.Runtime`类中，就用到了饿汉式的单例模式，相关代码如下所示：

![image-20211224204131281](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211224210636.png) 

###### 1.2.3.2 内部类懒汉式的体现

在jdk的`java.util.Collections`类中，有很多内部类懒汉式单例的体现，比如下面这个：

![image-20211224210341440](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211224210654.png) 

### 2.线程并发篇

#### 2.1 线程状态

##### 2.1.1 线程的六种状态

java线程的六种状态分别是：`NEW`，`RUNNABLE`，`TERMINATED`，`BLOCKED`，`WAITING`，`TIMED_WAITING`。

这六种状态之间的转换关系如下图所示：

![image-20211225154808442](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235239.png) 

图解如下：

`NEW`(新建)状态：

- 当一个线程对象被创建，但是还未调用start方法时处于**新建**状态；
- 该状态下还未与操作系统底层线程相关联。

`RUNNABLE`(可运行)状态：

- 调用了start方法后，就会从**新建**状态变成**可运行**状态；
- 进入该状态后便会与底层线程相关联，由操作系统调度执行。

`TERMINATED`(终结)状态：

- 线程内代码已经执行完毕，就会由**可运行**状态进入**终结**状态；
- 该状态下会取消与底层线程的关联。

`BLOCKED`(阻塞)状态：

- 当获取锁失败后，就会由可运行状态进入Monitor的阻塞队列**阻塞**，此时不占用CPU时间；
- 当持锁线程释放锁时，会按照一定规则唤醒阻塞队列中的**阻塞**线程，唤醒后的线程会进入**可运行**状态。

`WAITING`(等待)状态：

- 虽然成功获取到了锁，但是由于条件不满足，调用了wait()方法，此时会从**可运行**状态释放锁进入Monitor等待集合**等待**，同样不占用CPU时间；
- 当其他持锁线程调用notify()或notifyAll()方法后，会按照一定规则唤醒等待集合中的**等待**线程，恢复为**可运行**状态。

`TIMED_WAITING`(有时限等待)状态：

* 虽然成功获取到了锁，但是由于条件不满足，调用了wait(long)方法，此时会从**可运行**状态释放锁进入Monitor等待集合进行**有时限等待**，同样不占用CPU时间；
* 当其它持锁线程调用notify()或notifyAll()方法，会按照一定规则唤醒等待集合中的**有时限等待**线程，恢复为**可运行**状态，并重新去竞争锁；
* 如果等待超时，也会从**有时限等待**状态恢复为**可运行**状态，并重新去竞争锁；
* 还有一种情况，就是调用sleep(long)方法也会从**可运行**状态进入**有时限等待**状态，但与Monitor无关，不需要主动唤醒，超时时间到自然恢复为**可运行**状态。

> **说明**：我们也可以用interrupt()方法打断**等待**、**有时限等待**的线程，让它们恢复为**可运行**状态。

##### 2.1.2 代码演示

我们首先创建一个名为`t1`的线程，然后在调用其start()方法前后均打印出该线程的状态，代码如下：

```java
package com.interview.base.test;

/**
 * @Author: gongsl
 * @Date: 2021-12-22 20:42
 */
public class TestMain {
    public static void main(String[] args) {
        Thread t = new Thread(() -> {
            System.out.println("running...");
        }, "t1");
        System.out.println("线程名称：" + t.getName() + "，线程状态1：" + t.getState());
        t.start();
        System.out.println("线程名称：" + t.getName() + "，线程状态2：" + t.getState());
    }
}
```

运行以上代码后，控制台打印如下：

![image-20211225164314316](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235257.png) 

我们也可以通过debug的方式演示`t1`线程的`TERMINATED`状态，如下所示：

1. 在t1线程和主线程中均打上断点：

   ![image-20211225165214515](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235317.png) 

2. 断点类型均选择**Thread**：

   ![image-20211225165409585](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235325.png) 

3. 使用debug方式运行代码：

   ![image-20211225170227262](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235348.png)  

   ![image-20211225170035639](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235355.png) 

4. 最后观察控制台打印：

   ![image-20211225170510295](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235403.png) 

> **说明**：其他状态都需要debug方式才能演示出来，这里就不再进行演示了。

##### 2.1.3 关于五种状态

五种状态的说法来自于操作系统层面的划分，如下图所示：

![image-20210831092652602](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235412.png) 

- `新建态`与`终结态`：与前面提到的六种状态中同名状态类似；
- `就绪态`：表示有资格分到CPU时间，只是还未轮到它；
- `运行态`：表示已经分到CPU时间，能真正执行线程内代码；
- `阻塞态`：表示没资格分到CPU时间，阻塞态包含了`阻塞I/O`、`BLOCKED`、`WAITING`、`TIMED_WAITING`。

#### 2.2 线程池的核心参数

![image-20211225175114183](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235421.png) 

线程池主要有七大核心参数：

1. `corePoolSize`(核心线程数目)：池中会保留的最多线程数。
2. `maximumPoolSize`(最大线程数目)：核心线程 + 救急线程的最大数目。
3. `keepAliveTime`(生存时间)：救急线程的生存时间，生存时间内没有新任务，此线程资源会释放；
4. `unit`(时间单位)：救急线程的生存时间单位，如秒、毫秒等。
5. `workQueue`(阻塞队列)：当没有空闲核心线程时，新来任务会加入到此队列排队，队列满会创建救急线程执行任务。
6. `threadFactory`(线程工厂)：可以定制线程对象的创建，例如设置线程名字、是否是守护线程等。
7. `handler`(拒绝策略)：当所有线程都在繁忙，workQueue也放满时，会触发拒绝策略：
   - 抛异常：java.util.concurrent.ThreadPoolExecutor.AbortPolicy；
   - 由调用者执行任务：java.util.concurrent.ThreadPoolExecutor.CallerRunsPolicy；
   - 丢弃任务：java.util.concurrent.ThreadPoolExecutor.DiscardPolicy；
   - 丢弃最早排队任务：java.util.concurrent.ThreadPoolExecutor.DiscardOldestPolicy。

代码演示线程池的使用：

```java
package com.interview.base.test;

import java.util.concurrent.ArrayBlockingQueue;
import java.util.concurrent.ThreadPoolExecutor;
import java.util.concurrent.TimeUnit;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author: gongsl
 * @Date: 2021-12-25 22:02
 */
public class TestThreadPool {
    public static void main(String[] args){
        AtomicInteger c = new AtomicInteger(1);
        ThreadPoolExecutor threadPool = new ThreadPoolExecutor(2, 3, 0,
                TimeUnit.MILLISECONDS,
                new ArrayBlockingQueue<>(2),
                r -> new Thread(r, "myThread-" + c.getAndIncrement()),
                new ThreadPoolExecutor.AbortPolicy());
        
        threadPool.submit(() -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("线程名1：" + threadName);
        });
        threadPool.submit(() -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("线程名2：" + threadName);
        });
        threadPool.submit(() -> {
            String threadName = Thread.currentThread().getName();
            System.out.println("线程名3：" + threadName);
        });

    }
}
```

运行以上代码后，控制台打印如下：

![image-20211225224346603](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235449.png) 

> **说明**：以上代码中`corePoolSize`参数配置为2，所以线程池中只有两个活跃线程，所以使用三次submit()方法的时候，第三个线程的名称不是**myThread-3**而是**myThread-1**，说明第三个submit中的内容还是由第一个线程处理的。

#### 2.3 wait方法和sleep方法

##### 2.3.1 两者的异同

**共同点：**

- wait()，wait(long)和sleep(long)的效果都是让当前线程暂时放弃CPU的使用权，进入阻塞状态。

**不同点：**

- 方法归属不同：
  - sleep(long)是Thread的静态方法；
  - wait()，wait(long)都是Object的成员方法，每个对象都有。
- 醒来的时机不同：
  - 执行sleep(long)和wait(long)的线程都会在等待相应毫秒后醒来；
  - wait(long)和wait()还可以被notify唤醒，wait()如果不唤醒就会一直等下去。
- 锁的特性不同：
  * wait方法的调用必须先获取wait对象的锁，而sleep方法则无此限制；
  * wait方法执行后会释放对象锁，允许其它线程获得该对象锁；
  * 而sleep如果在synchronized代码块中执行，并不会释放对象锁。

##### 2.3.2 代码演示效果

wait方法是不能直接使用的，否则会抛异常，如下所示：

![image-20211225231853513](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235501.png) 

但是如果获取到锁之后再使用wait方法就没问题了，代码如下：

```java
package com.interview.base.test;

public class TestWaitVsSleep {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        synchronized (LOCK) {
            System.out.println("waiting...");
            LOCK.wait(5000);
        }
    }
}
```

wait方法执行后会释放对象锁，允许其它线程获得该对象锁，代码演示如下：

```java
package com.interview.base.test;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class TestWaitVsSleep {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            synchronized (LOCK) {
                try {
                    log.debug("waiting...");
                    LOCK.wait(5000);
                } catch (InterruptedException e) {
                    log.error("exception...");
                    e.printStackTrace();
                }
            }
        }, "t1");
        thread.start();

        //这里睡100毫秒是为了让异步线程先获取到锁
        Thread.sleep(100);
        synchronized (LOCK) {
            log.info("main...");
        }
    }
}
```

运行以上代码后，控制台打印如下：

![image-20211225233050104](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235521.png) 

> **说明**：通过控制台日志打印的时间可以发现，两条日志并不是差5秒，说明使用wait方法后确实会释放锁，并且允许其它线程获得该锁。但是如果将上面代码中的`LOCK.wait(5000)`换成`Thread.sleep(5000)`的话，控制台打印的两条日志就会相差5秒多了，因为在synchronized代码块中执行sleep方法，并不会释放锁。

无论是使用的wait方法还是sleep方法，都可以使用interrupt方法打断，以使用sleep方法为例演示，代码如下：

```java
package com.interview.base.test;

import lombok.extern.slf4j.Slf4j;

@Slf4j
public class TestWaitVsSleep {
    private static final Object LOCK = new Object();

    public static void main(String[] args) throws InterruptedException {
        Thread thread = new Thread(() -> {
            synchronized (LOCK) {
                try {
                    log.debug("waiting...");
                    Thread.sleep(5000);
                } catch (InterruptedException e) {
                    log.error("exception...");
                    e.printStackTrace();
                }
            }
        }, "t1");
        thread.start();

        //这里睡100毫秒是为了让异步线程先获取到锁
        Thread.sleep(100);
        thread.interrupt();
        synchronized (LOCK) {
            log.info("main...");
        }
    }
}
```

运行以上代码后，控制台打印如下所示：

![image-20211225234750755](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235534.png) 

> **说明**：上面的代码中使用了`Thread.sleep(5000)`方法，但是主线程和异步线程之间的执行时间是差100毫秒并不是5秒，说明使用interrupt()方法生效了，而且通过异常日志也可以看出来。

#### 2.4 lock和synchronized













### 3.JVM篇





### 4.Mysql篇





### 5.Redis篇





### 6.Linux篇





### 7.Spring篇





### 8.微服务篇





### 9.网络协议篇





### 10.算法篇





### 11.人事篇

































