### 1.基础篇

#### 1.1 List集合和Map集合

##### 1.1.1 List集合

###### 1.1.1.1 ArrayList的扩容机制

**ArrayList底层会维护一个Object[]数组，该数组初始大小取决于使用的构造方法：**

1. `ArrayList()`：使用无参构造方法创建ArrayList，底层会创建一个长度为零的数组；
2. `ArrayList(int initialCapacity)`：使用该构造方法的话，底层会创建一个指定容量的数组；
3. `ArrayList(Collection<? extends E> c)`：使用该构造方法，会将入参中集合的大小作为底层数组的容量。

**使用`add(E e)`方法对ArrayList底层数组容量的影响：**

当我们使用`new ArrayList()`创建一个集合时，底层默认会创建一个长度为零的数组，这时如果调用`add(E e)`方法向集合中添加元素，则会对底层数组进行扩容，第一次数组会扩容到==10==，不够的话第二次会扩容到15，第三次是22，每次容量的大小大约是前一次的1.5倍，之所以说是大约，是因为底层不是使用`size * 1.5`的方式，而是使用移位运算的方式，具体源码如下：

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
3. 尾部插入、删除性能可以，其它部位插入、删除都要复制并移动数据(比如在头部插入一个元素，那么集合中所有元素都要复制一份，然后每个都后移一位，以便空出头部位置给新的元素)，因此性能较低；
4. 可以利用CPU缓存，局部性原理，将内存中的数据加载进缓存，从而提升访问的性能。

**LinkedList：**

1. 基于双向链表，无需连续内存；
2. 随机访问慢(要沿着链表遍历)；
3. 头尾插入、删除性能高，其它部位插入、删除性能一般(因为中间部位需要根据链表一个个地移动指针去寻找，这个速度是比较慢的)。
4. LinkedList会比ArrayList占用更多的内存，因为LinkedList是由一个个的Node对象组成的，每个对象里面都包含了具体的元素以及上一个元素的指针信息和下一个元素的指针信息，所以每增加一个元素，都会占用不少内存。

![image-20211220210215781](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211220221541.png)

> **说明**：LinkedList是不能通过CPU缓存等方式提升性能的，因为其底层结构是链表，不是连续的内存。如果数据之间内存地址差太远的话，就算部分加载进缓存里了，也可能没缓存到多少有用的数据；而缓存的大小也是有限的，都加载进缓存的话，就可能把之前加载的覆盖掉。

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
* hash值如果足够随机，则在hash表内按泊松分布，在扩容因子为0.75的情况下，底层的Node数组中某个长度超过8的链表出现概率是0.00000006，树化阈值选择8就是为了让树化几率足够小。

**退化的规则：**

* 情况1：在扩容时如果拆分树，树元素个数<=6时，会退化至链表。
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
2. 大于这个值，空间节省了，但链表就可能会比较长，影响性能；
3. 小于这个值，冲突减少了，但扩容就会更频繁，空间占用也更多。

> **说明**：举例说明一下扩容因子的用法。比如数组的容量是16，扩容因子是0.75，两者相乘的结果就是12，那么当向数组中添加的元素个数超过12个时，就会触发数组扩容，即数组容量扩到32。而32和0.75相乘是24，所以当元素个数超过24时，数组又会扩容，即数组容量扩到64，依次类推。

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

> **注意**：使用双检锁懒汉式时，必须使用==volatile==进行修饰。因为`INSTANCE = new Singleton()`创建对象的操作不是原子的，主要分成三步：创建对象、调用构造方法、给静态变量赋值。其中后两步可能被指令重排序优化，从而变成先给静态变量赋值、再调用构造。多线程场景下，可能出现线程1执行了赋值，但还没来得及调用构造方法，然后线程2这时就执行到getInstance()方法中的第一个`INSTANCE == null`了。由于线程1已经给INSTANCE变量赋值了，所以线程2就会直接return，但是线程1还没调用构造方法，假设构造方法中是给一些别的成员变量赋值，那么就会出现线程2返回的单例对象中的一些成员变量没值的情况，而使用volatile就是为了避免指令重排序带来的这个问题。

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

#### 1.3 String字符串

##### 1.3.1 String的不可变性

String字符串是具有不可变性的，值一旦被确定，便不会发生改变，如果重新赋值，其实就是创建一个新的字符串。下面举例说明String的不可变性，演示代码如下：

```java
package com.interview.base.test;

public class TestString {

    String str = "abc";
    char[] arr = {'d','e'};

    public static void main(String[] args) {
        TestString ts = new TestString();
        ts.test(ts.str,ts.arr);
        System.out.println(ts.str);//运行结果：abc
        System.out.println(ts.arr);//运行结果：ce
    }

    public void test(String str, char[] arr){
        str = "ok";
        arr[0] = 'c';
    }
}
```

##### 1.3.2 字符串常量池

###### 1.3.2.1 StringTable的特点

- 字符串常量池就是StringTable，又称String pool，在java8中，字符串常量池是存在`堆内存`中的；
- StringTable中存储的并不是String类型的对象，而是指向String对象的索引，真实对象还是存储在堆中；
- 字符串常量池中是不会存储相同内容的字符串的。

###### 1.3.2.2 字符串的拼接

字符串的拼接分为常量与常量的拼接、常量与变量的拼接以及变量与变量的拼接，主要有以下特点：

- 常量与常量的拼接结果是在字符串常量池中，原理是编译器的优化；
- 字符串的拼接中只要有一个是变量，结果就在堆中，变量拼接的原理是使用了StringBuilder。

下面进行一个案例演示，代码如下：

```java
package com.interview.base.test;

public class TestString {
    public static void main(String[] args) {
        String s1 = "a";
        String s2 = "b";
        String s3 = "ab";
        String s4 = "a" + "b";
        String s5 = s1 + s2;
        String s6 = s1 + "b";
        System.out.println(s3 == s4);//运行结果：true
        System.out.println(s3 == s5);//运行结果：false
        System.out.println(s5 == s6);//运行结果：false
    }
}
```

**解释说明：**

- 在`String s3 = "ab"`这一步已经把"ab"放入到字符串常量池中了，`String s4 = "a" + "b"`这一步其实就是到字符串常量池中去取"ab"，所以s3和s4最终是一样的；
- 而`String s5 = s1 + s2`这一步就涉及到了变量和变量的拼接，原理其实是使用了StringBuilder，最终再调用其toString()方法返回结果，而toString()方法里面其实是使用new String()创建新的字符串对象，所以对s5赋值这一步相当于就是执行了new String("ab")操作；
- 只要字符串的拼接中有一个是变量，其实底层就会使用到StringBuilder，所以`String s6 = s1 + "b"`这一步其实也会创建一个新的字符串对象，等号比较的是地址，s5和s6是属于不同的字符串对象，所以地址肯定是不一样的。

###### 1.3.2.3 字符串对象问题

**String str = new String("ab")共创建了多少个对象？**

答案是`两个`，使用new关键字会创建一个，字符串常量池中也会创建一个，对应字节码文件内容如下：

![image-20220103141837856](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220103143635.png) 

**String str = new String("a") + new String("b")共创建了多少个对象？**

答案是`六个`，涉及变量的字符串拼接，首先会创建StringBuilder对象，通过new关键字会创建两个字符串对象，然后在字符串常量池中针对a和b又会创建两个对象，最终结果会通过StringBuilder的toString()方法返回，该方法内部使用new String()又创建了一个字符串对象，所以一共是六个。具体字节码文件内容如下：

![image-20220103142854974](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220103143642.png) 

> **说明**：通过`String str = new String("a") + new String("b")`会往字符串常量池中添加"a"、"b"两个字符串常量，但是不会添加"ab"。

#### 1.4 其他基础知识点

##### 1.4.1 关于面向对象编程的理解

**面向对象编程**，英文简称OOP，是一种程序设计思想。它把对象作为程序的基本单元，一个对象中包含了数据和操作数据的方法，`继承`、`封装`和`多态`是面向对象的三大特征。如果把面向过程比作是动作的执行者的话，那么面向对象就是动作的指挥者。面向对象编程侧重点是对象，它的核心思想是找合适的对象去做适合的事。

##### 1.4.2 java中的基本数据类型

- **整型：**
  - `byte`：占1个字节，位数为8位；
  - `short`：占2个字节，位数为16位；
  - `int`：占4个字节，位数为32位；
  - `long`：占8个字节，位数为64位。
- **浮点型：**
  - `float`：占4个字节，位数为32位；
  - `double`：占8个字节，位数为64位。
- **字符型：**
  - `char`：占2个字节，位数为16位。
- **布尔型：**
  - `boolean`：占1个字节，位数为8位。

##### 1.4.3 jdk、jre、jvm的区别

- `jdk`是java的开发工具包，它包含jre和一些好用的小工具，比如javac、jconsole等；
- `jre`是java的运行时环境，它主要包含jvm和一些核心类库；
- `jvm`是java虚拟机，它可以把class字节码文件解析成能够被计算机识别的指令，jvm是java能够跨平台的核心。

##### 1.4.4 强制类型转换的问题

**"short s1=1;s1=s1+1;"和short s1=1;s1+=1;"有什么区别？**

- 前者中的`s1+1`的运算结果是int型，需要进行强制类型转换，这里没有进行强制类型转换，所以会编译报错；
- 后面中的`s1+=1`没有问题，可以编译成功。

##### 1.4.5 int和Integer的区别

- Integer是int的包装类；
- Integer是类，默认值是null，int是基本数据类型，默认值是0；
- Integer表示的是对象，用一个引用指向这个对象，而int是基本数据类型，直接存储数值。

##### 1.4.6 抽象类和接口的区别

- 抽象类有构造方法，但是接口没有；
- java是单继承多实现的，一个类可以实现多个接口，但只能继承一个抽象类；
- 抽象类中的非抽象方法都是要有方法体的，而接口中的方法一般都是没有方法体的；
- 抽象类的设计目的是代码的复用，而接口的设计目的更多地是对类的行为进行约束。

##### 1.4.7 "=="和equals的区别

- 如果"=="比较的是基本数据类型的话，则比较的是数值是否相等；如果比较的是引用数据类型的话，则比较的是对象的地址是否相等；
- equals方法比较的是两个对象的内容是否相等，如果没有重写equals方法的话，比较的就是对象的地址是否相等。

##### 1.4.8 重载和重写的区别

- 在一个类中存在两个或者两个以上的同名函数，称为方法的重载；
- 子类中出现了与父类中名称和形参完全相同的方法，称为方法的重写，重写的前提是必须要存在继承关系。

##### 1.4.9 break、return、continue的区别

- break用于结束一个循环；
- return用于结束一个函数；
- continue则用于跳过本次循环，执行下次循环。

##### 1.4.10 super和this的区别

- `代表的事物不一致`：super关键字代表的是父类空间的引用，this关键字则代表所属函数的调用者对象；
- `使用前提不一致`：super关键字必须要有继承关系才能使用，this关键字不需要存在继承关系也可以使用；
- `调用的构造函数不一致`：super关键字是调用父类的构造函数，this关键字则是调用本类的构造函数。

##### 1.4.11 StringBuffer和StringBuilder的区别

- 两个类都是字符串缓冲类，它们的方法都是一致的，初始大小都是16字符；
- StringBuffer是线程安全的，操作效率低，StringBuilder是线程不安全的，操作效率高；
- StringBuffer是jdk1.0的时候出现的，而StringBuilder则是jdk1.5的时候出现的。

##### 1.4.12 Error和Exception的区别

- 错误(Error)一般是由JVM或者硬件引发的问题，这类问题一般无法通过代码去解决；
- 异常(Exception)则是可以通过代码去解决的。

##### 1.4.13 throws和throw的区别

- 有throws的时候可以没有throw，但是有throw的时候，如果throw抛的异常是Exception体系，那么必须要有throws在方法上声明；
- throws用于方法的声明上，其后跟的是异常类名，后面可以跟多个异常类，之间用逗号隔开；
- throw则用于方法体中，其后跟的是一个异常对象名。

##### 1.4.14 常见的RuntimeException异常

- `NullPointerException`：空指针异常；
- `ClassNotFoundException`：找不到指定类异常；
- `NumberFormatException`：字符串转换为数字异常；
- `IndexOutOfBoundsException`：数组角标越界异常；
- `ClassCastException`：数据类型转换异常；
- `IllegalArgumentException`：方法传递参数异常；
- `SQLException`：SQL异常；
- `NoSuchMethodException`：方法不存在异常。

##### 1.4.15 final、finally、finalize的区别

- final用于声明属性、方法和类，分别表示属性不可变，方法不可覆盖，类不可继承；
- finally是异常处理语句结构的一部分，不管是否有异常，finally块中的代码都会执行；
- finalize是Object类的一个方法，在垃圾收集器执行的时候会调用被回收对象的此方法。

##### 1.4.16 泛型中extends和super的区别

- `<? extends T>`表示包括T在内的任何T的子类；
- `<? super T>`表示包括T在内的任何T的父类；

##### 1.4.17 transient关键字的作用

- transient关键字一般在实现了Serializable接口的类中使用；
- transient关键字只能修饰变量，而不能修饰方法和类，局部变量是不能使用transient修饰的；
- 被transient关键字修饰的变量不参与序列化和反序列化；
- 一个静态变量无论是否被transient关键字修饰，该字段都不能被序列化。

##### 1.4.18 遇到过哪些设计模式

- `单例模式`：在jdk源码的java.util.Collections类中，就有很多内部类懒汉式单例的体现；
- `工厂模式`：Spring中的BeanFactory就是一种工厂模式的实现；
- `代理模式`：Mybatis中用到了jdk动态代理来生成Mappper的代理对象，在执行代理对象的方法时会去执行SQL。除此之外，Spring的AOP以及@Configuration注解的底层实现也都用到了代理模式；
- `责任链模式`：Tomcat中的Pipeline实现，以及Dubbo中的Filter机制都使用了责任链模式；
- `适配器模式`：Spring中的Bean销毁的生命周期中用到了适配器模式，用来适配各种Bean销毁逻辑的执行方式；
- `外观模式`：Tomcat中的Request和RequestFacade之间体现的就是外观模式；
- `模板方法模式`：Spring中的refresh方法中提供给子类继承重写的方法，就用到了模板方法模式。

##### 1.4.19 深拷贝和浅拷贝的区别

深拷贝和浅拷贝就是指对象的拷贝。一个对象中存在两种类型的属性，一种是基本数据类型，一种是实例对象的引用。

`浅拷贝`是指，只会拷贝基本数据类型的值，以及实例对象的引用地址，并不会复制一份引用地址所指向的对象。也就是浅拷贝出来的对象，内部的类属性指向的和拷贝前的是同一个对象。

`深拷贝`是指，不仅会拷贝基本数据类型的值，也会对实例对象的引用地址所指向的对象进行复制，所以对于深拷贝出来的对象，其内部的类属性指向的对象和拷贝前的不是同一个对象。

##### 1.4.20 mybatis中#{}和${}的区别

- \#{}是预编译处理、是占位符，${}是字符串替换、是拼接符；
- Mybatis在处理#{}时，会将sql中的#{}替换为问号，然后调用PreparedStatement的方法来赋值；
- Mybatis在处理${}时，则是直接将${}替换成变量的值；
- 使用#{}可以有效地防止SQL注入，提高系统的安全性。

### 2.并发篇

#### 2.1 进程和线程

##### 2.1.1 进程和线程的区别

进程和线程主要有以下几点区别：

- 进程是资源分配的最小单位，线程是CPU调度的最小单位；
- 进程可以看做是独立应用，而线程则不能；
- 进程有独立的地址空间，相互不影响，线程只是进程的不同执行路径；
- 运行一个程序会产生一个进程，而一个进程至少包含一个线程。

##### 2.1.2 自定义线程的创建方式

自定义线程一般包含两种创建方式：

**方式一：**

1. 自定义一个类继承Thread类；
2. 重写Thread类的run方法，把自定义线程的任务代码写在run方法中；
3. 创建Thread的子类对象，然后调用start方法开始线程即可。

**方式二：**

1. 自定义一个类实现Runnable接口；
2. 实现Runnable接口的run方法，把自定义线程的任务代码写在run方法中；
3. 创建Runnable实现类对象；
4. 创建Thread类对象，并且把Runnable实现类对象作为实参传递；
5. 调用Thread对象的start方法开始线程即可。

##### 2.1.3 线程状态

###### 2.1.3.1 线程的六种状态

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

###### 2.1.3.2 代码演示

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

###### 2.1.3.3 关于五种状态

五种状态的说法来自于操作系统层面的划分，如下图所示：

![image-20210831092652602](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211225235412.png) 

- `新建态`与`终结态`：与前面提到的六种状态中同名状态类似；
- `就绪态`：表示有资格分到CPU时间，只是还未轮到它；
- `运行态`：表示已经分到CPU时间，能真正执行线程内代码；
- `阻塞态`：表示没资格分到CPU时间，阻塞态包含了`阻塞I/O`、`BLOCKED`、`WAITING`、`TIMED_WAITING`。

#### 2.2 线程池相关知识点

##### 2.2.1 线程池的核心参数

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

##### 2.2.2 线程池的作用

使用线程池管理线程，主要有以下好处和作用：

- 通过重复利用线程池中已创建的线程可以降低线程创建和销毁造成的消耗；
- 使用线程池的话，当任务到达时，任务可以不需要等到线程创建就能立即执行，提高了响应速度；
- 使用线程池可以提高线程的可管理性；
- 使用线程池可以限定线程的个数，从而预防由于线程过多导致系统运行缓慢或崩溃的情况。

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

#### 2.4 synchronized关键字

##### 2.4.1 lock和synchronized

###### 2.4.1.1 两者的异同

**语法层面：**

* synchronized是关键字，源码在jvm中，用c++语言实现；
* Lock是接口，源码由jdk提供，用java语言实现；
* 使用synchronized时，退出同步代码块锁会自动释放，而使用Lock时，需要手动调用unlock方法释放锁；

**功能层面：**

* 二者均属于悲观锁、都具备基本的互斥、同步、锁重入功能；
* Lock提供了许多synchronized不具备的功能，例如获取等待状态、公平锁、可打断、可超时、多条件变量；
* Lock有适合不同场景的实现，如ReentrantLock、ReentrantReadWriteLock。

**性能层面：**

* 在没有竞争时，synchronized做了很多优化，如偏向锁、轻量级锁，性能不赖；
* 在竞争激烈时，Lock的实现通常会提供更好的性能。

###### 2.4.1.2 关于公平锁

* 公平锁的公平体现：
  * **已经处在阻塞队列**中的线程(不考虑超时)始终都是公平的，先进先出；
  * 公平锁是指**未处于阻塞队列**中的线程来争抢锁，如果队列不为空，则老实到队尾等待；
  * 非公平锁是指**未处于阻塞队列**中的线程来争抢锁，与队列头唤醒的线程去竞争，谁抢到算谁的。
* 公平锁会降低吞吐量，一般不用。

###### 2.4.1.3 关于条件变量

* ReentrantLock中的条件变量功能类似于普通synchronized的wait，notify，用在当线程获得锁后，发现条件不满足时，临时等待的链表结构；
* 与synchronized的等待集合不同之处在于，ReentrantLock中的条件变量可以有多个，可以实现更精细的等待、唤醒控制。

##### 2.4.2 ReentrantLock和synchronized

两者的主要区别如下所示：

- synchronized是一个关键字，而ReentrantLock是一个类；
- synchronized会自动地加锁与释放锁，ReentrantLock则需要手动加锁与释放锁；
- synchronized的底层是jvm层面的锁，ReentrantLock则是API层面的锁；
- synchronized是非公平锁，ReentrantLock则可以选择公平锁或非公平锁；
- synchronized底层有一个锁升级的过程。

##### 2.4.3 synchronized的几种锁

1. `偏向锁`：指使用synchronized把对象作为一把锁时，在锁对象的对象头中会记录当前获取到该锁的线程ID，该线程下次如果又来获取该锁，就可以直接获取到了；
2. `轻量级锁`：由偏向锁升级而来，当一个线程获取到锁后，此时这把锁是偏向锁，如果这时又有一个线程来竞争锁，偏向锁就会升级为轻量级锁。之所以叫轻量级锁，是为了和重量级锁进行区分，轻量级锁底层是通过自旋来实现的，并不会阻塞线程；
3. `重量级锁`：如果自旋次数过多仍然没有获取到锁，则会升级为重量级锁，重量级锁会导致线程阻塞；
4. `自旋锁`：自旋锁就是线程在获取锁的过程中，不会去阻塞线程，也就无所谓唤醒线程，阻塞和唤醒这两个步骤都是需要操作系统去进行的，比较消耗时间，自旋锁是线程通过CAS(乐观锁)获取预期的一个标记，如果没有获取到则继续循环获取，如果获取到了则表示获取到了锁，这个过程线程一直是在运行中的，相对而言没有使用太多的操作系统资源，所以是比较轻量的。

#### 2.5 volatile关键字

##### 2.5.1 线程安全的三个特性

线程安全要考虑三个方面：`可见性`、`有序性`、`原子性`。

- **可见性**：指一个线程对共享变量修改，另一个线程能看到最新的结果；
- **有序性**：指一个线程内代码按编写顺序执行；
- **原子性**：指一个线程内多行代码以一个整体运行，期间不能有其它线程的代码插队。

##### 2.5.2 三个特性的起因和解决办法

**可见性：**

* 起因：由于**编译器优化(比如JIT)、或缓存优化、或CPU指令重排序优化**导致的对共享变量所做的修改另外的线程看不到。
* 解决：用volatile修饰共享变量，能够防止编译器等优化发生，让一个线程对共享变量的修改对另一个线程可见。

**有序性：**

* 起因：由于**编译器优化、或缓存优化、或CPU指令重排序优化**导致指令的实际执行顺序与编写顺序不一致。
* 解决：用volatile修饰共享变量会在读、写共享变量时加入不同的屏障，阻止其他读写操作越过屏障，从而达到阻止重排序的效果。

**原子性：**

* 起因：多线程下，不同线程的**指令发生了交错**导致的共享变量的读写混乱。
* 解决：用悲观锁或乐观锁解决，volatile并不能解决原子性。

##### 2.5.3 volatile能否保证线程安全

volatile能够保证共享变量的**可见性**和**有序性**，但是并不能保证**原子性**。

注意事项：

* **volatile变量写**加的屏障是阻止上方其它写操作越过屏障排到**volatile变量写**之下；
* **volatile变量读**加的屏障是阻止下方其它读操作越过屏障排到**volatile变量读**之上；
* volatile读写加入的屏障只能防止同一线程内的指令重排。

#### 2.6 悲观锁和乐观锁

* 悲观锁的代表是synchronized和Lock锁：
  * 其核心思想是`线程只有占有了锁，才能去操作共享变量，每次只有一个线程占锁成功，获取锁失败的线程，都得停下来等待`；
  * 线程从运行到阻塞、再从阻塞到唤醒，涉及线程上下文切换，如果频繁发生，会影响性能；
  * 实际上，线程在获取到synchronized和Lock锁时，如果锁已被占用，都会先做几次重试操作，而不是直接进入阻塞队列，这样也是为了减少阻塞的可能。

* 乐观锁的代表是AtomicInteger，使用cas来保证原子性：
  * 其核心思想是`无需加锁，每次只有一个线程能成功修改共享变量，其它失败的线程不会停止，而是不断重试直至成功`；
  * 由于线程一直运行，不需要阻塞，因此不涉及线程上下文切换；
  * 它需要多核cpu支持，且线程数不应超过cpu核数。

#### 2.7 如何预防死锁

死锁发生的四个必要条件：

- `互斥条件`：同一时间只能有一个线程获取资源；
- `不可剥夺条件`：一个线程已经占有的资源，在释放之前不能被其它线程抢占；
- `请求和保持条件`：线程等待过程中不会释放已占有的资源；
- `循环等待条件`：多个线程互相等待对方释放资源。

只要破坏以上四个必要条件中的任意一个，就可以避免死锁的发生。

#### 2.8 Hashtable和ConcurrentHashMap

##### 2.8.1 两者的异同

* Hashtable与ConcurrentHashMap都是线程安全的Map集合；
* Hashtable并发度低，整个Hashtable对应一把锁，同一时刻，只能有一个线程操作它；
* ConcurrentHashMap并发度高，整个ConcurrentHashMap对应多把锁，只要线程访问的是不同锁，就不会冲突；
* 从jdk1.8开始，ConcurrentHashMap将数组的每个头节点作为锁，如果多个线程访问的头节点不同，则不会冲突。

> **说明**：从jdk1.8开始，ConcurrentHashMap底层结构也是采用的数组+链表+红黑树，所以数组中的每个元素其实都是头节点，然后每个元素下面根据实际情况可能会存在链表或者红黑树。

##### 2.8.2 jdk1.7中的ConcurrentHashMap

* 数据结构：`Segment(大数组) + HashEntry(小数组) + 链表`，每个Segment对应一把锁，如果多个线程访问不同的 Segment，则不会冲突；
* 并发度：Segment数组大小即并发度，决定了同一时刻最多能有多少个线程并发访问。Segment数组不能扩容，意味着并发度在ConcurrentHashMap创建时就固定了；
* 扩容：每个小数组的扩容相对独立，小数组元素个数超过`小数组容量 * 扩容因子`时会触发扩容，每次扩容翻倍；
* Segment[0]原型：首次创建其它小数组时，会以此原型为依据，数组长度，扩容因子都会以原型为准。

##### 2.8.3 jdk1.8中的ConcurrentHashMap

* 数据结构：`Node数组 + 链表 + 红黑树`，数组的每个头节点作为锁，如果多个线程访问的头节点不同，则不会冲突。首次生成头节点时如果发生竞争，会利用cas而非syncronized，进一步提升性能；
* 并发度：Node数组有多大，并发度就有多大，与jdk1.7的Segment(大数组)不同，jdk1.8的Node数组可以扩容；
* 扩容条件：Node数组中的元素个数达到数组容量的3/4时就会扩容；
* 扩容过程：以数组中每个元素的链表为单位，从数组最大的索引处开始，从后向前迁移数组每个索引处对应的链表，迁移完成的会将旧数组头节点替换为ForwardingNode，代表Node数组中的这个节点已经迁移完成。当数组中的所有节点都被替换为ForwardingNode时，扩容就算结束了；
* 扩容时并发get：
  * 根据是否为ForwardingNode来决定是在新数组查找还是在旧数组查找，不会阻塞；
  * 如果链表长度超过1，则需要对节点进行复制，即创建新节点；
  * 如果链表最后几个元素扩容后索引不变，则无需创建新的节点对象，会直接将原来的放到新数组中使用，这也是一种优化方式。
* 扩容时并发put：
  * 如果put的线程与扩容线程操作的链表是同一个，put线程会阻塞；
  * 如果put的线程操作的链表还未迁移完成，即头节点不是ForwardingNode，则可以并发执行；
  * 如果put的线程操作的链表已经迁移完成，即头节点是ForwardingNode，则可以协助扩容。
* 源码中的capacity属性值代表的是预估的元素个数，不是数组的初始大小；
* loadFactor只在计算初始数组大小时被使用，之后扩容因子固定为0.75；
* 超过树化阈值时的扩容问题，如果容量已经是64，直接树化，否则在原来容量基础上做3轮扩容。

#### 2.9 ThreadLocal的用法

##### 2.9.1 ThreadLocal的作用

ThreadLocal主要有以下两个作用：

* ThreadLocal可以实现`资源对象`的线程隔离，让每个线程各用各的`资源对象`，避免因争用引发的线程安全问题；
* ThreadLocal同时实现了线程内的资源共享。

下面我们通过代码的方式演示以上两个作用。首先创建5个线程，然后让它们都去获取数据库连接，如下所示：

```java
package com.interview.base.test;

import lombok.extern.slf4j.Slf4j;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

@Slf4j
public class TestThreadLocal {
    private static final ThreadLocal<Connection> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        //多个线程调用, 得到的都是自己的Connection对象
        for (int i = 0; i < 5; i++) {
            new Thread(() -> log.debug("{}", getConnection()), "thread-" + (i + 1)).start();
        }
    }

    public static Connection getConnection() {
        //到当前线程获取资源
        Connection conn = threadLocal.get();
        if (conn == null) {
            try {
                //创建新的连接对象
                String url = "jdbc:mysql://localhost:3306/test?useSSL=false";
                conn = DriverManager.getConnection(url, "root", "root");
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
            //将资源存入当前线程
            threadLocal.set(conn);
        }
        return conn;
    }
}
```

控制台打印内容如下：

![image-20211226175913884](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211226234559.png) 

> **说明**：由以上控制台打印结果可知，5个线程获取到的数据库的连接对象都不一样。这也正是ThreadLocal的第一个作用，实现了资源对象的线程隔离，让每个线程各用各的资源对象。

接下来我们演示ThreadLocal的第二个作用，即线程内的资源共享，代码如下所示：

```java
package com.interview.base.test;

import lombok.extern.slf4j.Slf4j;
import java.sql.Connection;
import java.sql.DriverManager;
import java.sql.SQLException;

@Slf4j
public class TestThreadLocal {
    private static final ThreadLocal<Connection> threadLocal = new ThreadLocal<>();

    public static void main(String[] args) {
        //多个线程调用, 得到的都是自己的Connection对象
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                log.debug("{}", getConnection());
                log.debug("{}", getConnection());
            }, "thread-" + (i + 1)).start();
        }
    }

    public static Connection getConnection() {
        //到当前线程获取资源
        Connection conn = threadLocal.get();
        if (conn == null) {
            try {
                //创建新的连接对象
                String url = "jdbc:mysql://localhost:3306/test?useSSL=false";
                conn = DriverManager.getConnection(url, "root", "root");
            } catch (SQLException e) {
                throw new RuntimeException(e);
            }
            //将资源存入当前线程
            threadLocal.set(conn);
        }
        return conn;
    }
}
```

以上代码我们运行了两个异步线程去获取数据库连接，并且每个线程都获取了两次，控制台打印如下：

![image-20211226181341596](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211226234609.png) 

> **说明**：通过上图可以发现，不同线程获取到的数据库连接对象虽然不一样，但是相同线程获取多次得到的数据库连接对象都是一样的，说明ThreadLocal确实实现了线程内的资源共享。

##### 2.9.2 ThreadLocal的原理

在ThreadLocal的底层其实是通过内部类`ThreadLocalMap`来存储资源对象的：

- 调用ThreadLocal类的set方法时，其实就是以ThreadLocal自己作为key，资源对象作为value，放入当前线程的ThreadLocalMap集合中；
- 调用get方法时，就是以ThreadLocal自己作为key，到当前线程中查找关联的资源值；
- 调用remove方法时，就是以ThreadLocal自己作为key，移除当前线程关联的资源值。

**关于ThreadLocalMap的一些特点：**

- key的hash值统一分配；
- 初始容量是16，扩容因子为2/3，扩容后容量翻倍；
- key索引冲突后用开放寻址法解决冲突；
- ThreadLocalMap中的key被设计为弱引用，当key不再使用的时候，如果内存不足，便于释放其占用的内存。

##### 2.9.3 内存释放时机

- ThreadLocalMap中的key被设计为弱引用，所以不再使用的时候是可以被释放的，不过仅仅会让key的内存释放，关联的value的内存并不会得到释放；
- 我们可以通过调用get方法释放value内存，因为如果key已经被释放，使用get方法时就会发现key值为null，这时候就会释放其value的内存；
- 或者我们也可以使用set方法通过启发式扫描释放value内存，如果key值已经被释放，再使用set方法，不仅会释放这个key对应value的内存，也会清除临近key值为null的value的内存；
- 我们也可以主动使用remove方法释放key、value的内存：
  - 使用remove方法不仅可以释放key、value的内存，也会清除临近key值为null的value的内存；
  - 推荐使用remove方法释放内存，因为我们一般使用ThreadLocal时，都是把它作为静态变量(即强引用)使用，因此无法被动依靠gc回收。

### 3.JVM篇

#### 3.1 JVM内存结构

##### 3.1.1 内存结构划分

下图就是JVM的内存结构划分图：

![image-20211226223454514](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211226234616.png) 

接下来将结合一段java代码的执行流程来理解jvm的内存结构划分，具体代码如下：

```java
public class TestMain {
    public static void main(String[] args) {
        Student stu = new Student();
        stu.study();
        stu.hashCode();
        stu = null;
    }
}
```

以上代码的详细执行流程如下：

- 以上代码属于java的源代码，对应图中的`Java Source`，源代码是不能被java虚拟机执行的，所以需要执行javac命令编译源代码为字节码，对应图中的`Java Class`，字节码文件是可以被java虚拟机识别和执行的；
- 然后执行java命令就会创建JVM，有了JVM之后就会调用`类加载子系统`加载class文件，将类的信息存入`方法区`，这些信息包括类的名称、类的继承关系、类的成员变量、类中方法的代码信息等等；
- 创建main主线程。使用的内存区域是JVM的栈内存，对应上图的`JVM Stacks虚拟机栈`，然后开始执行main方法代码；
- 由于Student类之前没见过，所以会继续触发类加载过程，Student类的信息同样会存入到`方法区`中；
- 一旦我们使用了new关键字创建对象，那么这个对象的信息就会被存到堆内存中，所以对象是占用的`堆`的内存；
- 像stu这种局部变量以及args这种方法参数所使用的都是当前线程的栈内存，即上图的`JVM Stacks虚拟机栈`；
- 调用方法时，像study()这种自己实现的方法属于普通方法，而hashCode()这种需要借助操作系统提供的一些函数去实现的属于本地方法。普通方法是存储到`JVM Stacks虚拟机栈`中的，本地方法则是存到`Native Method Stacks本地方法栈`中的，但是我们现在用的都是Oracle的Hotspot虚拟机实现，这个实现是不区分虚拟机栈和本地方法栈的，相当于是没有本地方法栈，所以最终这些方法都是存储在`JVM Stacks虚拟机栈`中，而且方法内的局部变量、方法参数等所使用的都是`JVM Stacks虚拟机栈`中的内存；
- 调用方法时，先要到`方法区`获得到该方法的字节码指令，由`解释器`将字节码指令解释为机器码执行；
- 调用方法时，会将要执行的指令行号读到`程序计数器`，这样当发生了线程切换，恢复时就可以从中断的位置继续；
- 对于频繁调用的热点方法或者频繁的循环代码，会由`JIT 即时编译器`将这些代码编译成机器码缓存，提高执行性能；
- 当我们创建的对象不再被任何地方引用的时候，这些对象就可以被当成是垃圾，当内存不足时，就会触发`垃圾回收`来回收掉这些对象占用的内存。

> **说明**：上面提到的`堆`和`方法区`是属于线程共享的，所以当一个线程的信息放在这里后，别的线程也可以访问到；但是上面的`程序计数器`以及`JVM Stacks虚拟机栈`则是属于线程私有的，别的线程是访问不到的。

##### 3.1.2 关于内存溢出

前面对内存结构做了详细介绍，那么哪些部分可能出现内存溢出呢？其实除了`程序计数器`外，像`方法区`、`堆`以及`虚拟机栈`都有可能出现内存溢出的情况。

内存溢出又分为==OutOfMemoryError==以及==StackOverflowError==这两种错误类型。

出现**OutOfMemoryError**的情况：

- `堆内存耗尽`：对象越来越多，又一直在使用，不能被垃圾回收。
- `方法区内存耗尽`：加载的类越来越多，很多框架都会在运行期间动态产生新的类。
- `虚拟机栈累积`：每个线程最多会占用1M内存，线程个数越来越多，而又长时间运行不销毁时。

出现**StackOverflowError**的区域：

- `JVM虚拟机栈`：原因有方法递归调用未正确结束、反序列化json时循环引用等。

> **说明**：每次方法调用都会占用该线程的内存，当该线程内的内存(最大1M)耗尽时，就会出现StackOverflowError错误。

##### 3.1.3 方法区及其实现

`方法区`是JVM规范中定义的一块内存区域，用来存储类元数据、方法字节码、即时编译器需要的信息等。而`永久代`则是在jdk1.8之前Hotspot虚拟机对JVM规范的实现，也就是对方法区的实现，在jdk1.8之后实现就变成了`元空间`，元空间使用本地内存作为方法区中信息的存储空间。

> **说明**：方法区只是一种规范或者定义，而永久代和元空间则是不同版本的jdk对方法区的实现。

由于永久代是老版本的jdk中方法区的实现，这里就不再详细介绍了，下面主要介绍下**元空间**的工作原理：

![image-20211229165031709](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232417.png) 

图解说明：

- 当第一次用到某个类时，由类加载器将class文件的类元信息读入，并存储到元空间中；
- 以上Student和Person的类元信息存储于元空间中，我们是无法直接访问的；
- 我们可以使用Student.class和Person.class间接访问类的元信息，它们属于java对象，可以在代码中使用；
- 在堆内存中，当类加载器下的所有类对象、实例对象都没有人引用时，gc就会对它们占用的内存进行释放；
- 类加载器下的所有对象的内存都被释放后，就会释放该类加载器占用的内存；
- 当发现堆内存中类加载器的内存被释放后，对应的元空间中的所有类元信息的内存也会被释放。

#### 3.2 JVM内存参数

##### 3.2.1 堆内存-按大小设置

![image-20211229213457104](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232440.png) 

**解释说明：**

- `-Xms`：最小堆内存(包括新生代和老年代)；
- `-Xmx`：最大堆内存(包括新生代和老年代)；
- `-Xmn`：新生代大小。

**注意事项：**

1. 上图中的**new**就是指新生代，**old**就是指老年代；
2. 上图中的**保留**是指，一开始不会占用那么多内存，随着使用内存越来越多，会逐步使用这部分保留内存；
3. 上图中的**-XX:NewSize**与**-XX:MaxNewSize**是用于设置新生代的最小值与最大值的，但一般不建议设置，由JVM自己控制就行；
4. 通常建议将**-Xmx**和**-Xms**的值设置一样，即不需要保留内存，不需要从小到大增长，这样性能更好；
5. 设置**-Xmn**相当于同时设置了**-XX:NewSize**与**-XX:MaxNewSize**，并且两者取值相等。

##### 3.2.2 堆内存-按比例设置

![image-20211229214954300](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232447.png) 

**解释说明：**

- `-XX:NewRatio=2:1`：表示将堆内存分为三份，老年代占两份，新生代占一份；
- `-XX:SurvivorRatio=4:1`：表示将新生代分成六份，伊甸园(eden)占四份，from和to各占一份.

> **说明**：新生代由eden、from、to三者组成，其中from和to大小默认是一样的。并且默认情况下，eden和from的比例是8:1。

##### 3.2.3 元空间内存设置

![image-20211229221021785](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232452.png) 

**解释说明：**

- `class space`：用于存储类的基本信息，比如类的名称等，最大值受`-XX:CompressedClassSpaceSize`控制；
- `non-class space`：用于存储除类的基本信息以外的其它信息，比如方法字节码、注解等；
- **class space**和**non-class space**总大小受`-XX:MaxMetaspaceSize`控制。

> **注意**：这里`-XX:CompressedClassSpaceSize`这段空间默认大小是1G，并且它还与是否开启了指针压缩有关，这里暂不深入展开，可以简单认为指针压缩默认开启。

##### 3.2.4 代码缓存区内存设置 

![image-20211229222517516](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232458.jpg) 

**解释说明：**

- 如果`-XX:ReservedCodeCacheSize` < 240m，所有优化机器代码将会不加区分，都存在一起；
- 否则，分成三个区域：
  - `non-nmethods`：JVM自己用的代码；
  - `profiled nmethods`：部分优化的机器码；
  - `non-profiled nmethods`：完全优化的机器码。

> **说明**：前面有提到说`JIT 即时编译器`会将热点代码编译成机器码后进行缓存以提高效率，编译后的机器码其实就是缓存到这个CodeCache代码缓存区的。

##### 3.2.5 线程内存设置

我们可以通过`-Xss`参数为jvm启动的线程分配内存大小，该参数等同于`-XX:ThreadStackSize`，在不同操作系统下该参数的默认值不同，具体可以通过[官网](https://docs.oracle.com/javase/8/docs/technotes/tools/unix/java.html)进行查看，比如在64位的Linux系统下默认值是1M，如下图所示：

![image-20211229232046034](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20211229232505.png) 

#### 3.3 JVM垃圾回收

##### 3.3.1 垃圾回收算法

垃圾回收总共有`标记清除法`、`标记整理法`、`标记复制法`三种算法，这三种算法都有**标记**这个词，这里标记的意思其实就是找到那些不能被垃圾回收的对象，然后对它们进行标记。将来进行垃圾回收的时候，被标记的内容就会被保留，没有被标记的就会被清理掉，然后释放其占用的内存。

###### 3.3.1.1 标记清除法

![image-20220102130111482](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231905.png) 

**图解说明：**

- 首先会找到GC Root对象(根对象)，即那些一定不会被回收的对象，比如正在执行方法内局部变量引用的对象、静态变量引用的对象等；
- 然后在标记阶段，会沿着GC Root对象的引用链进行寻找，被根对象直接或间接引用到的对象都会被加上标记；
- 最后在清除阶段，会释放掉未加标记的对象占用的内存(比如上图的灰色区域)。

**注意事项：**

- 由于未加标记而被清除的对象很大可能都是不连续的，所以如果这部分内存被释放了，剩下的被标记的对象之间很大可能也都会不连续，这就会产生很多内存碎片。这种缺陷的最直接体现就是，明明内存充足，但是由于内存都不连续，所以当我们想要创建一个需要连续内存的对象时(比如数组)，可能会因为没有足够的连续内存而创建失败。

> **说明**：由于标记清除法存在比较明显的内存碎片缺陷，所以这种算法在新版的jdk中已经不被使用了。

###### 3.3.1.2 标记整理法

![image-20220102133929653](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231915.png) 

**图解说明：**

- 标记清除阶段和标记清除法是类似的，都是先标记，然后再清除未被标记的对象；
- 不过标记整理法最后还会多一步整理的动作，即将被标记的对象向一端移动，使之形成连续内存，避免产生内存碎片。

**注意事项：**

- 标记整理法可以有效避免内存碎片的问题，但是由于多了一步整理的动作，所以性能会有所下降。

###### 3.3.1.3 标记复制法

![20220102134811](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231921.jpg) 

**图解说明：**

- 标记复制法会将内存分成两个大小相等的区域，即from区域和to区域，from区域用于存储新创建的对象，to区域刚开始的时候是空闲的，什么也不存；
- 然后将被标记的对象从from区域复制到to区域，复制的过程中自然完成了碎片整理；
- 复制完成后，清除from区域的所有内容，最后交换from区域和to区域的位置即可。

**注意事项：**

- 从from区域复制到to区域的过程，是一个挨着一个复制的，所以复制到to区域的内容肯定都是连续的，自然就没有内存碎片的问题；
- 标记复制法不需要整理，它是标记后直接进行复制，复制完成后直接清除掉from区域的所有内容，所以效率要比标记整理法高；
- 但是由于标记复制法需要两个大小相等的内存区域，所以这种方式的缺点是会占用成倍的内存空间；

> **说明**：标记复制法和标记整理法在垃圾回收器中都有被使用，不过标记复制法经常用于新生代的垃圾回收，因为新生代的存活对象一般较少，这样复制的工作量就少，效率就高。而老年代的存活对象可能会比较多，垃圾对象比较少，所以一般适合使用标记整理法。

##### 3.3.2 GC与分代回收算法

###### 3.3.2.1 GC

GC的目的在于实现无用对象内存自动释放，减少内存碎片，加快分配速度。

**GC要点：**

* 回收区域是`堆内存`，不包括虚拟机栈，在方法调用结束会自动释放方法占用内存；
* 判断无用对象，使用`可达性分析算法`，`三色标记法`标记存活对象，回收未标记对象；
* GC具体的实现称为`垃圾回收器`；
* GC大都采用了`分代回收思想`(分代分的就是新生代和老年代)：
  * 理论依据是大部分对象朝生夕灭，用完立刻就可以回收，另有部分对象会长时间存活，每次很难回收；
  * 根据这两类对象的特性将回收区域分为**新生代**和**老年代**，新生代采用标记复制法、老年代一般采用标记整理法。
* 根据GC的规模可以分成**Minor GC**，**Mixed GC**，**Full GC**。

**GC规模：**

- `Minor GC`：发生在新生代的垃圾回收，暂停用户线程时间短；
- `Mixed GC`：新生代+老年代部分区域的垃圾回收，G1收集器特有；
- `Full GC`：新生代+老年代完整垃圾回收，暂停用户线程时间长，**应尽量避免**。

###### 3.3.2.2 分代回收

1. 伊甸园eden，最初对象都分配到这里，与幸存区survivor(分成from和to)合称`新生代`，如下图：

   ![image-20220102165805642](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231929.png) 

2. 当伊甸园内存不足时，会标记伊甸园与from(现阶段没有)的存活对象，比如下面的黑色部分：

   ![image-20220102170159438](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231933.png) 

3. 然后将存活对象复制到to中，复制完毕后，伊甸园和from内存都得到释放：

   ![image-20220102170347431](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231941.png) 

4. 之后将from和to交换位置：

   ![image-20220102170621205](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231946.png) 

5. 假设经过一段时间后伊甸园的内存又出现不足：

   ![image-20220102170730185](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231953.png) 

6. 标记伊甸园与from的存活对象：

   ![image-20220102170947974](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102231958.png) 

7. 将存活对象复制到to中：

   ![image-20220102171046413](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232003.png) 

8. 复制完毕后，伊甸园和from内存都得到释放：

   ![image-20220102171220035](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232009.png) 

9. 最后将from和to交换位置：

   ![image-20220102171502913](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232014.png) 

10. 以上都是新生代垃圾回收的场景，当幸存区对象熬过几次回收(最多15次)后，就会晋升到老年代，当幸存区内存不足或有大对象时可能会提前晋升至老年代。

##### 3.3.3 三色标记和并发漏标

###### 3.3.3.1 三色标记

在标记对象时，采用的是三色标记法，即使用三种颜色记录对象的标记状态：

- `黑色`：表示已标记；
- `灰色`：表示标记中；
- `白色`：表示还未标记。

> **说明**：被标记为黑色的对象即为不被垃圾回收的对象，而为白色的对象在垃圾回收时可以被当做垃圾回收掉。

###### 3.3.3.2 并发漏标问题

比较先进的垃圾回收器都支持**并发标记**，即在标记过程中，用户线程仍然能工作。但这样带来一个新的问题，如果用户线程修改了对象引用，那么就存在漏标问题。例如已经被标记为黑色的对象又有了新的引用，这时候这个新的引用就会被漏标。

因此对于**并发标记**而言，必须解决漏标问题，也就是要记录标记过程中的变化。目前有两种解决方法：

1. `Incremental Update`：增量更新法，CMS垃圾回收器采用。思路是拦截每次赋值动作，只要赋值发生，被赋值的对象就会被记录下来，在重新标记阶段再确认一遍。
2. `Snapshot At The Beginning(SATB)`：原始快照法，G1垃圾回收器采用。思路也是拦截每次赋值动作，不过记录的对象不同，也需要在重新标记阶段对这些对象二次处理。

**注意事项：**

- 增量更新法记录的是已经被标记为黑色的对象，而原始快照法记录的是新加的对象，以及被删除引用关系的对象；
- 在解决并发漏标问题时，无论是使用哪些解决方法，在重新标记阶段，都需要STW(Stop The World)，即暂停整个应用程序线程。

##### 3.3.4 垃圾回收器

上文有提到过，GC具体的实现称为**垃圾回收器**，而这里就介绍几种垃圾回收器。

###### 3.3.4.1 Parallel GC

**Parallel GC特点如下：**

* 新生代eden内存不足时发生Minor GC，采用标记复制算法，需要暂停用户线程；
* 老年代内存不足时发生Full GC，采用标记整理算法，需要暂停用户线程；
* 对于注重吞吐量的应用程序，适合使用Parallel GC。

###### 3.3.4.2 ConcurrentMarkSweep GC

**ConcurrentMarkSweep GC特点如下：**

* 它是工作在老年代的垃圾回收器，是支持**并发标记**的一款回收器，采用标记清除算法：
  * 并发标记时不需暂停用户线程；
  * 重新标记时仍需暂停用户线程。

* 如果并发失败(即回收速度赶不上创建新对象速度)，会触发Full GC；

* 对于注重响应时间的应用程序，适合使用ConcurrentMarkSweep GC。

> **说明**：在最新的jdk版本中，这种垃圾回收器已经被废弃了。

###### 3.3.4.3 G1 GC

**G1 GC特点如下：**

- 这种垃圾回收器是一款**响应时间**与**吞吐量**兼顾的垃圾回收器；
- 它被划分成多个区域，每个区域都能充当eden，survivor，old，humongous，其中humongous专为大对象准备；
- 主要有三个阶段，即新生代回收、并发标记、混合收集；
- 如果并发失败(即回收速度赶不上创建新对象速度)，会触发Full GC。

> **说明**：从java9开始，G1 GC已经被作为默认垃圾回收器使用了。

#### 3.4 内存溢出的场景

##### 3.4.1 误用线程池导致的内存溢出

演示代码如下所示：

```java
package com.interview.base.test;

import lombok.extern.slf4j.Slf4j;
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.TimeUnit;

/**
 * @Author: gongsl
 * @Date: 2022-01-02 19:17
 */
@Slf4j
public class TestOomThreadPool {
    public static void main(String[] args) {
        ExecutorService executor = Executors.newFixedThreadPool(2);
        log.debug("begin...");
        while (true) {
            executor.submit(() -> {
                try {
                    log.info("send msg...");
                    TimeUnit.SECONDS.sleep(30);
                }catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
    }
}
```

为了在较短时间内就出现报错，这里把最大堆内存设置小一点，比如`-Xmx16m`，如下所示：

![image-20220102193549543](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232025.png) 

运行代码后，控制台最终打印内容如下：

![image-20220102193652494](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232039.png) 

如果想要知道出现内存溢出的原因，那就要看看`Executors.newFixedThreadPool(int nThreads)`方法底层了，如下：

![image-20220102194033140](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232045.png) 

通过上图可以发现，底层创建的队列并没有设置上限，而我们在代码中又在源源不断地往队列中添加一段时间内无法被处理的任务，所以最终任务对象所耗费的内存导致了堆内存的耗尽。

> **注意**：使用`Executors.newCachedThreadPool()`也有可能会出现内存溢出，这里就不再进行详细介绍了。

##### 3.4.2 查询数据量太大导致的内存溢出

比如电商业务，每件商品都有商品详情，如果商品详情内容很多，详情接口调用量又很大，就有可能导致内存溢出。解决的办法是，数据库层面在进行查询的时候，最好不要find all，比如我们可以使用limit限制一下每次sql查询的数量。

##### 3.4.3 动态生成类导致的内存溢出

如果项目中使用到了会动态生成类的工具，一定要注意其用法，避免动态生成类过多，导致元空间内存溢出。

#### 3.5 关于类加载

##### 3.5.1 类加载过程

关于类加载过程的三个阶段：

1. **加载：**
   - 将类的字节码载入方法区，并创建类`.class`对象(对象存在堆内存中)；
   - 如果此类的父类没有加载，先加载父类；
   - 加载是懒惰执行，即用到才加载。

2. **链接：**
   - 验证：验证类是否符合Class规范，合法性、安全性检查；
   - 准备：为static变量分配空间，设置默认值；
   - 解析：将常量池的符号引用解析为直接引用。

3. **初始化：**
   - 静态代码块、static修饰的变量赋值、static final修饰的引用类型变量赋值，会被合并成一个方法，在初始化时被调用；
   - static final修饰的基本类型变量赋值，在链接阶段就已完成；
   - 初始化是懒惰执行，即用到才初始化。

##### 3.5.2 类加载器

jdk的版本不同，类加载器可能会有不同，所以这里就以java8为例，讲解该版本的类加载器，如下所示：

| 名称                    | 加载哪的类            | 说明               |
| ----------------------- | --------------------- | ------------------ |
| Bootstrap ClassLoader   | JAVA_HOME/jre/lib     | 无法直接访问       |
| Extension ClassLoader   | JAVA_HOME/jre/lib/ext | 上级为 Bootstrap   |
| Application ClassLoader | classpath             | 上级为 Extension   |
| 自定义类加载器          | 自定义                | 上级为 Application |

##### 3.5.3 双亲委派机制

所谓的双亲委派，就是指优先委派上级类加载器进行加载，如果上级类加载器

* 能找到这个类，由上级加载，加载后该类也对下级加载器可见；
* 找不到这个类，则下级类加载器才有资格执行加载。

双亲委派的目的有两点：

1. 让上级类加载器中的类对下级共享，即能让我们的类能依赖到jdk提供的核心类；
2. 让类的加载有优先次序，保证核心类优先加载。

> **注意**：下级类加载器中的类对上级是不共享的。

#### 3.6 对象的四种引用类型

对象的引用类型主要包括：`强引用`、`软引用`、`弱引用`、`虚引用`这四种。

##### 3.6.1 强引用

1. 普通变量赋值即为强引用，如A a = new A()；
2. 通过GC Root的引用链，如果强引用不到该对象，该对象才能被回收。

![image-20220102222541073](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232102.png) 

##### 3.6.2 软引用(SoftReference)

1. 例如：SoftReference a = new SoftReference(new A())；
2. 如果仅有软引用该对象时，首次垃圾回收不会回收该对象，如果内存仍不足，再次回收时才会释放该对象；
3. 软引用自身需要配合引用队列来释放；
4. 典型例子是反射数据。

![image-20220102222655147](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232106.png) 

##### 3.6.3 弱引用(WeakReference)

1. 例如：WeakReference a = new WeakReference(new A())；

2. 如果仅有弱引用引用该对象时，只要发生垃圾回收，就会释放该对象；

3. 弱引用自身需要配合引用队列来释放；

4. 典型例子是ThreadLocalMap中的Entry对象。

![image-20210901112107707](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232110.png) 

##### 3.6.4 虚引用(PhantomReference)

1. 例如：PhantomReference a = new PhantomReference(new A(), referenceQueue)；

2. 必须配合引用队列一起使用，当虚引用所引用的对象被回收时，由Reference Handler线程将虚引用对象入队，这样就可以知道哪些对象被回收，从而对它们关联的资源做进一步处理；

3. 典型例子是Cleaner释放DirectByteBuffer关联的直接内存。

![image-20210901112157901](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220102232114.png) 

#### 3.7 关于finalize

**finalize的特点：**

* 它是Object中的一个方法，如果子类重写它，垃圾回收时此方法会被调用，可以在其中进行资源释放和清理工作；
* 不过将资源释放和清理放在finalize方法中非常不好，非常影响性能，严重时甚至会引起OOM，从Java9开始它就被标注为@Deprecated，不建议被使用了。

**finalize的缺点：**

- 无法保证资源释放，因为其底层的FinalizerThread是守护线程，代码很有可能没来得及执行完，线程就结束了；
- 无法判断是否发生错误，执行finalize方法时，其底层会吞掉任意异常(Throwable)；
- 内存释放不及时，重写了 finalize 方法的对象在第一次被 gc 时，并不能及时释放它占用的内存，因为要等着FinalizerThread调用完finalize，把它从unfinalized队列移除后，第二次gc时才能真正释放内存。

#### 3.8 JVM命令行工具

##### 3.8.1 jps命令

jps命令是用于显示java进程的相关信息的，主要包含以下几个常见参数：

- `-q`：只显示进程id；
- `-l`：输出应用程序主类的全类名，如果进程执行的是jar包，则输出jar包的完整路径；
- `-m`：输出虚拟机进程启动时传递给主类的参数；
- `-v`：累出虚拟机进程启动时的jvm参数，比如-Xms500m、-Xmx500m等。

![image-20220125182715916](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161509.png) 

##### 3.8.2 jstat命令

jstat命令有很多参数选项，这里主要介绍如下几种：

- ==-class==：显示ClassLoader的相关信息，比如`jstat -class 126496 1000 5`命令：

  ![image-20220125213311518](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161516.png) 

- ==-gc==：显示与GC相关的堆信息，比如`jstat -gc -t 126496 1000 5`命令：

  ![image-20220125214025926](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161522.png) 

- ==-gccause==：能输出导致最后一次或当前正在发生的GC产生的原因，比如`jstat -gccause 126496 1000 5`命令：

  ![image-20220126124802467](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161528.png) 

我们可以根据GCT列推断是否可能发生内存溢出。我们可以使用jstat命令每隔10秒打印一次记录，然后观察每条记录之间GCT列的差值，差值即表示GC总时间的增量，如果有差值大于2秒，说明GC的时间占运行时间的比例大于20%，这表明目前堆内存压力较大；如果两条记录之间的差值大于8秒，说明GC的时间占运行时间的比例大于80%，这意味着堆中几乎没有可用空间了，随时可能抛出OOM异常。

下面就演示下内存溢出的情况，并观察GCT列的数据变化。对应的java代码如下：

```java
/**
 * 模拟内存溢出
 */
@GetMapping("/memory/out")
public String leak() {
    log.info("模拟内存溢出...");
    ThreadLocal<Byte[]> localVariable = new ThreadLocal<Byte[]>();
    localVariable.set(new Byte[4096 * 1024]);// 为线程添加变量
    return "ok";
}
```

为了演示内存溢出，在启动java项目的时候先分配一下内存，如下所示：

```bash
java -jar -Xms500m -Xmx500m home-1.0-SNAPSHOT.jar
```

项目启动后，使用如下命令进行重复调用，增加压力，以便能够出现内存溢出的情况：

```bash
for i in {1..100}; do curl http://localhost:8080/memory/out;echo "";done
```

项目日志中出现如下报错，说明发生了内存溢出：

```bash
Handler dispatch failed; nested exception is java.lang.OutOfMemoryError: Java heap space
```

![image-20220126131532505](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161537.png) 

通过`jstat -gccause 71826 10000`命令(71826是java项目的进程id)实时观察GC的情况，每10秒展示一条记录。可以发现，在出现内存溢出的时候，Full GC次数和GC花费的时间陡然剧增，垃圾回收的时间已经，如下图：

![image-20220126134703770](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161545.png) 

##### 3.8.3 jmap命令

使用jmap命令可以生成堆转储快照文件，通过`-dump`参数即可生成对应的dump文件，命令如下：

```bash
# format=b表示生成的是二进制的文件，file后跟的是文件名，pid是java项目的进程id
jmap -dump:format=b,file=<filename> <pid>

# 下面多加一个live后，和上面命令的区别是，生成的dump文件中只会保存堆中的存活对象
jmap -dump:live,format=b,file=<filename> <pid>
```

比如使用`jmap -dump:live,format=b,file=/root/home/dumpfile.hprof 110568`命令生成一个dump文件：

![image-20220126154017580](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161553.png) 

如果我们想要在内存溢出时自动生成dump文件，可以在启动项目时，增加如下两个参数：

```bash
-XX:+HeapDumpOnOutOfMemoryError 
-XX:HeapDumpPath
```

![image-20220126155614235](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161602.png) 

然后通过某种方式让进程产生OOM异常，之后发现在当前路径下确实自动生成了我们指定文件名的dump文件，如下所示：

![image-20220126155856411](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161610.png) 

> **说明**：生产环境下，dump文件一般都会比较大，使用自动生成dump文件的方式可能会有文件，所以要慎用。

我们还可以使用`-heap`参数输出整个堆空间的详细信息，包括GC的使用、堆配置信息以及内存的使用信息等，命令如下：

```bash
# pid是java项目的进程id
jmap -heap <pid>
```

比如使用`jmap -heap 110568`命令获取对应进程id的整个堆空间的详细信息，如下所示：

![image-20220126155042765](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161616.png) 

##### 3.8.4 jstack命令

使用jmap命令可以生成堆转储快照，而jstack命令则用于生成虚拟机当前时刻的线程快照，线程快照就是当前虚拟机内每一条线程正在执行的方法堆栈的集合。

生成线程快照的目的主要是定位线程长时间出现停顿的原因，如线程间死锁、死循环、请求外部资源导致的长时间等待等都是导致线程长时间停顿的原因。线程出现停顿的时候通过`jstack`来查看各个线程的调用堆栈，就可以知道没有响应的线程到底在后台做些什么事情，或者在等待些什么资源。

jstack的常规使用方式就是`jstack <pid>`，pid就是我们项目的进程id，我们还可以使用`jstack -l <pid>`命令，使用`-l`参数后，除堆栈外，还可以显示关于锁的附加信息。为了深入理解jstack的用法，下面模拟两个异常的场景，并通过jstack命令去排查和定位原因。

###### 3.8.4.1 jstack排查死锁问题

下面通过java代码模拟线程死锁问题，对应的代码如下：

```java
/**
 * 模拟死锁
 */
@GetMapping("/thread/lock")
public String testLock() {
    log.info("模拟死锁...");
    new Thread(() -> {
        synchronized (resource1) {
            log.info(Thread.currentThread() + "get resource1");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info(Thread.currentThread() + "waiting get resource2");
            synchronized (resource2) {
                log.info(Thread.currentThread() + "get resource2");
            }
        }
    }, "thread-1").start();

    new Thread(() -> {
        synchronized (resource2) {
            log.info(Thread.currentThread() + "get resource2");
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            log.info(Thread.currentThread() + "waiting get resource1");
            synchronized (resource1) {
                log.info(Thread.currentThread() + "get resource1");
            }
        }
    }, "thread-2").start();
    return "ok";
}
```

通过`java -jar home-1.0-SNAPSHOT.jar`命令运行项目代码包后，可通过如下命令进行访问：

```bash
curl http://localhost:8080/thread/lock
```

访问成功后，其实基本就已经出现了死锁，先使用`jps -l`命令查看进程id，我这边的进程id是**26342**，然后我们就可以使用`jstack 26342`命令打印线程快照信息了，打印出来的部分线程快照信息如下：

```bash
2022-01-26 17:29:07
Full thread dump Java HotSpot(TM) 64-Bit Server VM (25.231-b11 mixed mode):

"Attach Listener" #32 daemon prio=9 os_prio=0 tid=0x00007f595c001000 nid=0x6a8c waiting on condition [0x0000000000000000]
   java.lang.Thread.State: RUNNABLE

"thread-2" #31 daemon prio=5 os_prio=0 tid=0x0000000002359000 nid=0x69d6 waiting for monitor entry [0x00007f59514d0000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.gongsl.controller.HelloController.lambda$testLock$1(HelloController.java:84)
        - waiting to lock <0x00000000e13ff788> (a java.lang.Object)
        - locked <0x00000000e13ff778> (a java.lang.Object)
        at com.gongsl.controller.HelloController$$Lambda$627/799446959.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

"thread-1" #30 daemon prio=5 os_prio=0 tid=0x0000000002350000 nid=0x69d5 waiting for monitor entry [0x00007f59515d1000]
   java.lang.Thread.State: BLOCKED (on object monitor)
        at com.gongsl.controller.HelloController.lambda$testLock$0(HelloController.java:69)
        - waiting to lock <0x00000000e13ff778> (a java.lang.Object)
        - locked <0x00000000e13ff788> (a java.lang.Object)
        at com.gongsl.controller.HelloController$$Lambda$626/247239777.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

......

Found one Java-level deadlock:
=============================
"thread-2":
  waiting to lock monitor 0x00007f59640062c8 (object 0x00000000e13ff788, a java.lang.Object),
  which is held by "thread-1"
"thread-1":
  waiting to lock monitor 0x00007f5964002178 (object 0x00000000e13ff778, a java.lang.Object),
  which is held by "thread-2"

Java stack information for the threads listed above:
===================================================
"thread-2":
        at com.gongsl.controller.HelloController.lambda$testLock$1(HelloController.java:84)
        - waiting to lock <0x00000000e13ff788> (a java.lang.Object)
        - locked <0x00000000e13ff778> (a java.lang.Object)
        at com.gongsl.controller.HelloController$$Lambda$627/799446959.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)
"thread-1":
        at com.gongsl.controller.HelloController.lambda$testLock$0(HelloController.java:69)
        - waiting to lock <0x00000000e13ff778> (a java.lang.Object)
        - locked <0x00000000e13ff788> (a java.lang.Object)
        at com.gongsl.controller.HelloController$$Lambda$626/247239777.run(Unknown Source)
        at java.lang.Thread.run(Thread.java:748)

Found 1 deadlock.
```

> **说明**：通过以上的线程快照信息可以发现，是**thread-1**和**thread-2**这两个线程发生了死锁。

###### 3.8.4.2 jstack排查CPU占满问题 

下面通过java代码模拟CPU占满问题，对应的代码如下：

```java
/**
 * 模拟CPU占满
 */
@GetMapping("/cpu/loop")
public void testCPULoop(){
    log.info("模拟CPU占满...");
    Thread.currentThread().setName("loop-thread-cpu");
    int num = 0;
    while (true) {
        num++;
        if (num == Integer.MAX_VALUE) {
            log.info("reset...");
        }
        num = 0;
    }
}
```

通过`java -jar home-1.0-SNAPSHOT.jar`命令运行项目代码包后，可通过如下命令进行访问：

```bash
curl http://localhost:8080/cpu/loop
```

访问成功后，使用`top`命令查看CPU的使用情况：

![image-20220127152319548](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161631.png) 

通过上图可以发现，进程**89951**对应的java应用的CPU已经飙到190%了，肯定是有问题的。下面再看下这个进程对应的线程情况，可以使用`top -Hp 89951`命令，结果如下所示：

![image-20220127152743799](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161637.png) 

接下来使用jstack命令查看线程dump快照中线程id为**89970**的线程正在执行的代码行，如下所示：

```bash
# 使用"printf "%x" 89970"命令是为了获取16进制的线程id
jstack 89951|grep -A5 $(printf "%x" 89970)
```

![image-20220127153131464](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220127161644.png) 

> **说明**：通过以上截图可知，已经定位出CPU占满的原因了，我们只需要对相应的代码进行问题分析即可。

### 4.Spring篇

#### 4.1 Spring的IOC和AOP

##### 4.1.1 Spring IOC

控制反转(Inversion of Control，缩写为`IOC`)就是我们平时说的IOC，IOC容器其实是一个Map集合，里面存放的都是各种对象，我们可以通过`@Component`注解等方式将对象加入到IOC容器中。当需要使用到容器中的对象时，可以再通过`@Autowired`等注解进行依赖注入(Dependency Injection，简称`DI`)。

在没有引入IOC之前，我们都是手动去创建依赖的对象，这个是正转的。在引入IOC之后，依赖的对象是直接由IOC容器创建后再注入到对象中，这就由主动创建变成了被动接受，这个是反转的，控制权颠倒过来了，所以就叫控制反转。

##### 4.1.2 Spring AOP 

AOP的全称是Aspect Oriented Programming，即`面向切面编程`，它是为解耦而生的，面向切面编程也可以说是对面向对象编程(OOP)的补充和完善。它底层使用了代理的方式，作用是在不改变原有业务代码逻辑的基础上，增加一些额外的功能。AOP可以将程序中交叉的业务逻辑(比如安全、日志、事务等)封装成一个切面，最后再注入到目标对象中。

#### 4.2 Spring的事务

##### 4.2.1 事务的特性

事务主要有以下四大特性：

- `原子性`(Atomicity)：强调事务的不可分割，事务中的操作要么都发生，要么都不发生；
- `一致性`(Consistency)：事务的执行前后数据的完整性应保持一致，数据不会因为事务的执行而遭到破坏；
- `隔离性`(Isolation)：一个事务的执行，不应该受到其他事务的干扰，即并发执行的事务之间互不干扰；
- `持久性`(Durability)：一个事务一旦提交，它对数据库中数据的改变就是永久性的。

> **说明**：事务的这四个特性简称ACID，数据库也有事务，都是满足这四个特性的。

##### 4.2.2 事务会产生的问题

| 名称         | 数据的状态 | 实际行为                                 | 产生原因   |
| ------------ | ---------- | ---------------------------------------- | ---------- |
| `脏读`       | 未提交     | 打算提交但是数据回滚了，读取了提交的数据 | 数据的读取 |
| `不可重复读` | 已提交     | 读取了修改前的数据                       | 数据的修改 |
| `幻读`       | 已提交     | 读取了插入前的数据                       | 数据的插入 |

##### 4.2.3 Spring事务的传播机制

Spring一共定义了七种事务传播机制：

1. `REQUIRED`(required)：如果当前没有事务，则新建一个事务，如果当前存在事务，则加入这个事务，这个是默认的传播机制。
2. `SUPPORTS`(supports)：当前存在事务，则加入当前事务，如果当前没有事务，则以非事务的方式执行。
3. `MANDATORY`(mandatory)：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
4. `REQUIRES_NEW`(requires_new)：总是创建一个新事务，如果当前已经存在事务，则挂起已经存在的事务。
5. `NOT_SUPPORTED`(not_supported)：以非事务方式执行，如果存在当前事务，则挂起当前事务。
6. `NEVER`(never)：不使用事务，如果当前存在事务，则抛出异常。
7. `NESTED`(nested)：如果当前存在事务，则在嵌套事务内执行，如果当前没有事务，则执行与**REQUIRED**一样的操作。

在java代码中，以上七种事务传播机制的使用案例如下：

```java
@Transactional(propagation = Propagation.REQUIRED)
@Transactional(propagation = Propagation.SUPPORTS)
@Transactional(propagation = Propagation.MANDATORY)
@Transactional(propagation = Propagation.REQUIRES_NEW)
@Transactional(propagation = Propagation.NOT_SUPPORTED)
@Transactional(propagation = Propagation.NEVER)
@Transactional(propagation = Propagation.NESTED)
```

**NESTED和REQUIRES_NEW的区别：**

`REQUIRES_NEW`是新建一个事务，并且新开始的这个事务和原有事务无关，而`NESTED`则是当前存在事务时会开启一个嵌套事务；在NESTED情况下，父事务回滚时，子事务也会回滚，而在REQUIRES_NEW情况下，原有事务回滚，不会影响新开启的事务。

##### 4.2.4 Spring事务的隔离级别

Spring事务主要有以下五种隔离级别：

- `DEFAULT`：这个是Spring默认的隔离级别，使用该隔离级别表示会以数据库的默认隔离级别为准，mysql数据库的默认隔离级别是**REPEATABLE_READ(可重复读)**，oracle数据库的是**READ_COMMITTED(读已提交)**；
- `READ_UNCOMMITTED`(读未提交)：一个事务可以感知或者操作另外一个未提交的事务，该级别可能会出现脏读、不可重复读、幻读等问题；
- `READ_COMMITTED`(读已提交)：一个事务只能感知或者操作另一个已经提交的事务，可能会出现不可重复读、幻读；
- `REPEATABLE_READ`(可重复读)：表示对同一字段的多次读取结果都是一致的，该级别能够避免脏读、不可重复读，但是不能避免幻读；
- `SERIALIZABLE`(串行化)：这个是最高的隔离级别，该隔离级别可以避免所有因并发导致的脏读、不可重复读、幻读等问题，但是性能较低。

如果我们项目中使用的是`@Transactional`注解的话，可以通过该注解中的参数设置传播机制和隔离级别，案例如下：

```java
@Transactional(propagation = Propagation.REQUIRES_NEW,isolation = Isolation.REPEATABLE_READ)
```

##### 4.2.5 Spring事务失效的场景

1. ==抛出检查异常导致事务失效==

   ```java
   @Service
   public class ServiceImpl {
   
       @Autowired
       private AccountMapper accountMapper;
   
       @Transactional
       public void transfer(int from, int to, int amount) throws FileNotFoundException {
           int fromBalance = accountMapper.findBalanceBy(from);
           if (fromBalance - amount >= 0) {
               accountMapper.update(from, -1 * amount);
               //abc这个文件是不存在的，所以会抛找不到文件的异常
               new FileInputStream("abc");
               accountMapper.update(to, amount);
           }
       }
   }
   ```

   **原因：**Spring默认只会回滚非检查异常，比如RuntimeException。

   **解决办法：**配置rollbackFor属性，比如：`@Transactional(rollbackFor = Exception.class)`，意思是对于所有异常都进行回滚。

2. ==try-catch掉异常导致事务失效==

   ```java
   @Service
   public class ServiceImpl {
   
       @Autowired
       private AccountMapper accountMapper;
   
       @Transactional(rollbackFor = Exception.class)
       public void transfer(int from, int to, int amount)  {
           try {
               int fromBalance = accountMapper.findBalanceBy(from);
               if (fromBalance - amount >= 0) {
                   accountMapper.update(from, -1 * amount);
                   new FileInputStream("aaa");
                   accountMapper.update(to, amount);
               }
           } catch (FileNotFoundException e) {
               e.printStackTrace();
           }
       }
   }
   ```

   **原因：**事务通知只有捕捉到了目标抛出的异常，才能进行后续的回滚处理，如果目标自己处理掉了异常，事务通知是无法知悉的，也就无法进行回滚。

   **解决办法：**在catch块中添加`throw new RuntimeException(e);`来抛出一个运行时异常，或者在catch块中添加如下内容也行，和抛出运行时异常效果是一样的：

   ```java
   TransactionInterceptor.currentTransactionStatus().setRollbackOnly();
   ```

3. ==AOP切面顺序问题导致事务失效==

   **现象：**业务代码中针对添加事务的方法做了自定义切面处理，但是在切面中将异常捕获掉了，导致异常无法回滚。

   **原因：**事务也是一个切面，该切面的默认优先级要比我们自定义切面的高，属于外层切面，所以业务代码中抛出异常后，首先会经过我们的自定义切面，在自定义切面中将异常捕获的话，外层的事务切面就感知不到异常了，自然也就无法执行回滚操作。

   **解决办法：**我们可以在自定义切面的catch块中再抛出一个运行时异常，让事务切面感知到异常，或者也可以通过使用`@Order`注解调高自定义切面的优先级，让事务切面变成内层切面，这样就可以优先处理业务异常。

4. ==非public方法导致事务失效==

   **原因：**Spring为方法创建代理、添加事务通知等的前提条件都是该方法要是public的。

   **解决办法**：

   - 将方法改为public方法即可。

   - 或者我们也可以通过添加如下bean配置来解决：

     ```java
     @Bean
     public TransactionAttributeSource transactionAttributeSource() {
         return new AnnotationTransactionAttributeSource(false);
     }
     ```

     > **注意**：一般不推荐使用这个添加bean配置的方式来解决事务失效的问题。

5. ==父子容器导致的事务失效==

   **原因：**子容器扫描范围过大，把未加事务配置的service也扫描进来了。

   **解决办法：**各自扫描各自的，或者干脆不要使用父子容器，所有bean都放在同一容器中。

   > **说明**：使用SpringBoot进行项目开发时是没有父子容器的，所以也不会出现这类事务失效的问题，一般在传统的Spring + MVC项目中才可能会出现父子容器的场景。

#### 4.3 SpringBoot

##### 4.3.1 SpringBoot简介

SpringBoot并不是一个框架，它是Spring生态的产品，它简化了使用Spring的难度，省略了繁重的xml配置，能够快速创建出生产级别的Spring应用。

##### 4.3.2 SpringBoot的优缺点

**SpringBoot的优点：**

1. 可以快速创建独立的Spring应用；
2. 内嵌了web服务器，无需再以war包形式部署项目；
3. 提供了一系列的starter启动器，简化了构建配置；
4. 自动版本仲裁功能可以让我们无需担心依赖的版本冲突问题；
5. 自动配置Spring以及第三方功能；
6. 提供生产级别的监控、健康检查及外部化配置；
7. 无代码生成、无需编写XML文件等。

**SpringBoot的缺点：**

1. 人称版本帝，迭代快，需要时刻关注变化；
2. 封装太深，内部原理复杂，不容易精通。

##### 4.3.3 SpringBoot中的常用注解

SpringBoot中的常用注解如下：

- `@SpringBootApplication`：这是一个组合注解，用在SpringBoot主类上，标识这是一个SpringBoot应用，用来开启SpringBoot的各项能力；
- `@EnableAutoConfiguration`：这是一个允许SpringBoot自动配置的注解，开启这个注解之后，SpringBoot就能根据当前类路径下的包或者类来配置Spring Bean；
- `@Configuration`：用来代替applicationContext.xml配置文件，所有这个配置文件里面能做到的事情都可以通过这个注解所在类来进行注册，添加该注解的类也叫配置类；
- `@ComponentScan`：开启组件扫描，即自动扫描包路径下的@Component注解进行注册bean实例到context中；
- `@Bean`：用来代替xml配置文件里面的bean配置；
- `@ImportResource`：对于有些通过类的注册方式配置不了的，可以通过这个注解引入额外的xml配置文件；
- `@Import`：可以使用这个注解给容器中自动创建多个不同类型的组件；

##### 4.3.4 SpringBoot自动装配原理

1. 通过`@SpringBootApplication`注解引入`@EnableAutoConfiguration`注解：

   ![image-20220216170922308](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220216170936.png)  

2. 然后再通过`@EnableAutoConfiguration`注解引入`@Import(AutoConfigurationImportSelector.class)`：

   ![image-20220105165444338](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231325.png) 

3. 在AutoConfigurationImportSelector类中有一个内部类AutoConfigurationGroup，其中有一个process方法，该方法中会调用到AutoConfigurationImportSelector类的getAutoConfigurationEntry方法：

   ![image-20220105170021949](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231333.png) 

4. 在getAutoConfigurationEntry方法中又会调用getCandidateConfigurations方法，然后一直深入：

   ![image-20220105170502265](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231339.png) 

   ![image-20220105170613569](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231347.png) 

   ![image-20220105172629698](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231354.png) 

   > **说明**：最终会读取所有模块中`META-INF/spring.factories`文件里的内容。

5. 以spring-boot-autoconfigure-2.3.7.RELEASE.jar中的**META-INF/spring.factories**文件为例，如果我们使用了`@EnableAutoConfiguration`注解，默认会加载spring.factories文件中对应的所有类：

   ![image-20220105175535495](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231403.png) 

   > **说明**：我这边只是截取了一部分，EnableAutoConfiguration对应的会被自动装配的类一共有127个，不同版本的SpringBoot可能会有不同。

6. 上一步的那些类并不会都被放到IOC容器中，因为每个类上都会有类似`@ConditionalOnClass`这样的注解，只有满足一定条件的自动配置类才会被放入到容器中。

### 5.Mysql篇

#### 5.1 数据库三大范式

为了建立冗余较小、结构合理的数据库，设计数据库时必须遵循一定的规则，在关系型数据库中这种规则就称为范式。范式是符合某一种设计要求的总结，要想设计一个结构合理的关系型数据库，必须满足一定的范式。

在实际开发中，最为常见的设计范式有三个：

- `第一范式`：要求表的每个字段必须是不可分割的独立单元；
- `第二范式`：在第一范式的基础上，要求每张表只表达一个意思，表的每个字段都和表的主键有依赖关系；
- `第三范式`：在第二范式的基础上，要求每张表的主键之外的其他字段都只能和主键有直接决定性的依赖关系。

#### 5.2 mysql索引

索引是帮助MySQL高效获取数据的数据结构(有序)。在数据之外，数据库系统还维护着满足特定查找算法的数据结构，这些数据结构以某种方式引用(指向)数据，这样就可以在这些数据结构上实现高级查找算法，这种数据结构就是`索引`。

一般来说，索引本身也很大，不可能全部存储在内存中，因此索引往往以索引文件的形式存储在磁盘上，所以索引查找过程中是会产生磁盘I/O消耗的。

##### 5.2.1 索引的设计原则

- 更新频繁的列不应设置索引；
- 数据量小的表不要使用索引；
- 重复数据多的字段不应设为索引(比如性别，只有男和女，一般来说，重复的数据超过百分之15就不该建索引)；
- 首先应该考虑对where和order by涉及的列上建立索引。

##### 5.2.2 索引的优缺点

**索引的优点：**

- 使用索引可以加快数据的查询速度，并且可以降低查询时对磁盘的I/O消耗；
- 索引可以帮助服务器避免排序和创建临时表；
- 通过使用索引，可以在查询的过程中，使用优化隐藏器，提高系统的性能。

**索引的缺点：**

- 索引也是需要占据磁盘空间的，所以索引并不是越多越好；
- 创建和维护索引都是需要消耗时间的，并且随着数据量的增加，时间也会增加；
- 虽然索引可以提高查询效率，但同时也会降低更新表的速度，因为当对表进行insert、update、delete操作时，需要动态更改索引文件中记录的表数据相关的信息。

##### 5.2.3 索引的分类

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231207.png" alt="image-20220106215332302" style="zoom: 60%;" /> 

##### 5.2.4 索引的结构

MySQL中有多种索引结构(索引底层的数据结构)，常用Hash，B树，B+树等数据结构来进行数据存储。这里只介绍下最常用的B树及其的变体B+树。

索引是在MySQL的存储引擎层中实现的，而不是在服务器层实现的。所以每种存储引擎的索引都不一定完全相同，也不是所有的存储引擎都支持所有的索引类型的。**我们常用的InnoDB引擎、MyISAM引擎默认都使用的是B+树索引**。

B树是一种多路搜索树，下图是3阶B树：

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231220.png" alt="image-20220106182432004" style="zoom: 65%;" /> 

**B树的特征：**

- 关键字集合分布在整棵树中；
- 任何一个关键字出现且只出现在一个节点中；
- 搜索有可能在非叶子节点结束；
- 其搜索性能等价于在关键字全集内做一次二分查找；
- 自动层次控制。

> **说明**：B树的搜索，从根节点开始，对节点内的关键字(有序)序列进行二分查找，如果命中则结束，否则进入查询关键字所属范围的孩子节点，依次重复，直到所对应的孩子指针为空或已经是叶子节点。

B+树是B树的变体，也是一种多路搜索树，如下图所示：

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231233.png" alt="image-20220106182503116" style="zoom: 76%;" /> 

**B+树的特征：**

- 所有关键字都出现在叶子节点的链表中(稠密索引)，且链表中的关键字恰好是有序的；
- 不可能在非叶子节点命中；
- 非叶子节点相当于是叶子节点的索引(稀疏索引)，叶子节点相当于是存储数据的数据层；
- 每一个叶子节点都包含指向下一个叶子节点的指针，从而方便叶子节点的范围遍历。

> **说明**：B+树的搜索与B树也基本相同，区别是B+树只有达到叶子节点才命中(B树可以在非叶子节点命中)，其性能也等价于在关键字全集做一次二分查找。

**B+树比B树更适合作为索引的原因：**

- `B+树的磁盘读写代价更低`：B+树的数据都集中在叶子节点，分支节点只负责指针(索引)。B树的分支节点既有指针也有数据，这将导致B+树的层高会小于B树的层高，也就是说B+树平均的I/O次数会小于B树；
- `B+树的查询效率更加稳定`：B+树的数据都存在叶子节点，故任何关键字的查找必须走一条从根节点到叶子节点的路径，所有关键字的查询路径相同，每个数据查询效率相当；
- `B+树更便于遍历`：由于B+树的数据都在叶子节点中，分支节点均为索引，所以遍历只需要扫描一遍叶子节点即可；B树因为其分支节点同样存储着数据，所以要找到具体的数据，遍历较B+树将更加麻烦；
- `B+树更擅长范围查询`：B+树叶子节点存放数据，数据是按顺序放置的双向链表，并且每一个叶子节点都包含指向下一个叶子节点的指针，所以更适合范围查询；
- `B+树占用内存空间小`：B+树索引节点没有数据，占用内存很小，所以在内存有限的情况下，相比于B树索引可以加载更多B+树索引。

**B+树索引和哈希索引的区别：**

- 如果是等值查询，那么哈希索引明显有绝对优势，因为只需要经过一次算法即可找到相应的键值；这有个前提，键值都是唯一的。如果键值不是唯一的，就需要先找到该键所在位置，然后再根据链表往后扫描，直到找到相应的数据；
- 如果是范围查询检索，这时候哈希索引就毫无用武之地了，因为原先是有序的键值，经过哈希算法后，有可能变成不连续的了，就没办法再利用索引完成范围查询检索；
- 哈希索引也没办法利用索引完成排序，以及like这样的模糊查询(这种模糊查询，其实本质上也是范围查询)；
- 哈希索引也不支持多列联合索引的最左匹配规则；
- B+树索引的关键字检索效率比较平均，不像B树那样波动幅度大，另外，在有大量重复键值情况下，哈希索引的效率也是极低的，因为存在所谓的哈希碰撞问题。

##### 5.2.5 聚簇与非聚簇索引

`聚簇`是为了提高某个属性(或属性组)的查询速度，把这个或这些属性(称为聚簇码)上具有相同值的元组集中存放在连续的物理块。

`聚簇索引`(clustered index)不是单独的一种索引类型，而是一种数据存储方式。这种存储方式是依靠B+树来实现的，根据表的主键构造一棵B+树且B+树叶子节点存放的都是表的行记录数据时，方可称该主键索引为聚簇索引。聚簇索引也可理解为将数据存储与索引放到了一块，找到索引也就找到了数据。

`非聚簇索引`(也叫辅助索引或二级索引)的数据和索引是分开的，B+树叶子节点存放的不是数据表的行记录，而是指针这些信息，然后再通过指针等信息找到对应的表的行记录，即表数据。

虽然InnoDB和MyISAM存储引擎都默认使用B+树结构存储索引，但是只有InnoDB的主键索引才是聚簇索引，InnoDB中的辅助索引以及MyISAM使用的都是非聚簇索引。一个表中只能存在一个聚簇索引(主键索引)，但可以存在多个非聚簇索引。

**聚簇索引的优缺点：**

**优点：**

- 数据访问更快，因为聚簇索引将索引和数据保存在同一个B+树中，因此从聚簇索引中获取数据比非聚簇索引更快；
- 聚簇索引对于主键的排序查找和范围查找速度非常快；

**缺点：**

- 插入速度严重依赖于插入顺序，按照主键的顺序插入是最快的方式，否则将会出现页分裂，严重影响性能。因此，对于InnoDB表，我们一般都会定义一个自增的ID列为主键，主键列最好不要选没有意义的自增列，选经常查询的条件列才好，不然无法体现其主键索引性能；
- 更新主键的代价很高，因为将会导致被更新的行移动，因此，对于InnoDB表，我们一般定义主键为不可更新。

> **说明**：网上还会有`密集索引`和`稀疏索引`的说法，密集索引就可以理解成是聚簇索引，而稀疏索引则可以理解成是非聚簇索引。

#### 5.3 mysql存储引擎

##### 5.3.1 入门概述

数据库存储引擎是数据库底层软件组织，数据库管理系统(DBMS)使用数据引擎进行创建、查询、更新和删除数据。不同的存储引擎提供不同的存储机制、索引技巧、锁定水平等功能。现在许多不同的数据库管理系统都支持多种不同的数据引擎，**MySQL的核心就是存储引擎**。用户可以根据不同的需求为数据表选择不同的存储引擎。

创建新表时如果不指定存储引擎，那么系统就会使用默认的存储引擎，MySQL5.5之前的默认存储引擎是**MyISAM**，5.5之后就改为了**InnoDB**。

我们可以使用`show engines;`命令查看mysql支持的存储引擎以及默认使用的存储引擎：

![image-20220107172710622](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220204201730.png) 

我们也可以直接使用`show variables like '%default_storage_engine%';`命令查看默认的存储引擎：

![image-20220107173017552](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220204201736.png) 

##### 5.3.2 常见的存储引擎

mysql支持的存储引擎有很多，下面只介绍常用的三种：

- `InnoDB`：事务型速记的首选引擎，支持ACID事务，支持行级锁定，mysql5.5之后的默认数据库引擎；
- `MyISAM`：拥有较高的插入、查询速度，但不支持事务，mysql5.5之前的默认数据库引擎；
- `Memory`：所有数据置于内存的存储引擎，拥有极高的插入、更新和查询效率，但是会占用和数据量成正比的内存空间，并且其内容会在Mysql重新启动时丢失。

**以上三种存储引擎特性的区别如下：**

| 特点         | InnoDB               | MyISAM   | MEMORY |
| ------------ | -------------------- | -------- | ------ |
| 存储限制     | 64TB                 | 256TB    | RAM    |
| 事务安全     | ==支持==             |          |        |
| 锁机制       | ==行锁(适合高并发)== | ==表锁== | 表锁   |
| B树索引      | 支持                 | 支持     | 支持   |
| 哈希索引     |                      |          | 支持   |
| 全文索引     | 支持(5.6版本之后)    | 支持     |        |
| 集群索引     | 支持                 |          |        |
| 数据索引     | 支持                 |          | 支持   |
| 索引缓存     | 支持                 | 支持     | 支持   |
| 数据可压缩   |                      | 支持     |        |
| 空间使用     | 高                   | 低       | N/A    |
| 内存使用     | 高                   | 低       | 中等   |
| 批量插入速度 | 低                   | 高       | 高     |
| 支持外键     | ==支持==             |          |        |

##### 5.3.3 三种存储引擎的特点

###### 5.3.3.1 InnoDB引擎

InnoDB是一个事务安全的存储引擎，它具备提交、回滚以及崩溃恢复的功能以保护用户数据。InnoDB的行级别锁定保证数据一致性，提升了它的多用户并发数以及性能。InnoDB将用户数据存储在聚集索引中以减少基于主键的普通查询所带来的I/O开销，为了保证数据的完整性，InnoDB还支持外键约束。另外，InnoDB默认使用B+树数据结构存储索引。

**InnoDB的特点：**

- 支持事务，支持4个事务隔离(ACID)级别；
- 支持行级锁定(更新时锁定当前行)；
- 读写阻塞与事务隔离级别相关；
- 既能缓存索引又能缓存数据；
- 支持外键；
- InnoDB更消耗资源，读取速度没有MyISAM快；
- 在InnoDB中存在着缓冲管理，通过缓冲池，将索引和数据全部缓存起来，加快查询的速度；
- 对于InnoDB类型的表，其数据的物理组织形式是聚簇表，所有的数据按照主键来组织。数据和索引放在一块，都位于B+树的叶子节点上；
- 对于InnoDB类型的表，主键会被作为聚簇索引，如果没有定义主键，则将该表第一个唯一非空索引作为聚簇索引；如果没有唯一非空的索引字段，InnoDB内部会生成一个隐藏主键作为聚簇索引。

###### 5.3.3.2 MyISAM引擎

MyISAM既不支持事务、也不支持外键，其优势是访问速度快，但是表级别的锁定限制了它在读写负载方面的性能，因此它经常应用于只读或者以读为主的数据场景。它也是默认使用B+树数据结构存储索引。

**MyISAM的特点：**

- 不支持事务；
- 表级锁定(更新时锁定整个表)；
- 读写互相阻塞(写入时阻塞读入、读时阻塞写入，但是读不会互相阻塞)；
- 只会缓存索引(通过key_buffer_size缓存索引，但是不会缓存数据)；
- 不支持外键；
- 读取速度快。

###### 5.3.3.3 Memory引擎

在内存中创建表。每个MEMORY表只实际对应一个磁盘文件(frm表结构文件)。MEMORY类型的表访问非常快，因为它的数据是放在内存中的，并且默认使用HASH索引。

**Memory的特点：**

- 支持的数据类型有限制，比如不支持TEXT和BLOB类型(长度不固定)，对于字符串类型的数据，只支持固定长度的行，VARCHAR会被自动存储为CHAR类型；
- 支持的锁粒度为表级锁。所以，在访问量比较大时，表级锁会成为MEMORY存储引擎的瓶颈；
- 由于数据是存放在内存中，所以一旦服务器出现故障，数据都会丢失；
- 查询的时候，如果有用到临时表，而且临时表中有BLOB，TEXT类型的字段，那么这个临时表就会转化为MyISAM类型的表，性能会急剧降低；
- 默认使用hash索引；
- 如果一个内部表很大，会转化为磁盘表。

#### 5.4 mysql锁

锁是计算机协调多个进程或线程并发访问某一资源的机制(避免争抢)。

在数据库中，除传统的计算资源(如CPU、RAM、I/O等)的争用以外，数据也是一种供许多用户共享的资源。如何保证数据并发访问的一致性、有效性是所有数据库必须解决的一个问题，锁冲突也是影响数据库并发访问性能的一个重要因素。从这个角度来说，锁对数据库而言显得尤其重要，也更加复杂。

##### 5.4.1 锁的分类

从对数据操作的粒度分： 

- `表锁`：操作时，会锁定整个表；
- `行锁`：操作时，会锁定当前操作行。

从对数据操作的类型分：

- `读锁`(共享锁)：针对同一份数据，多个读操作可以同时进行而不会互相影响；
- `写锁`(排它锁)：当前操作没有完成之前，它会阻断其他写锁和读锁。

##### 5.4.2 mysql的锁机制

数据库加锁的背景其实是事务并发问题，而mysql的默认隔离级别是**可重复读**，前面讲Spring事务的时候也有提到过，事务涉及的内容都是差不多的，所以这里就不再赘述了，我们也可以使用`show variables like '%tx_isolation%';`命令直接到mysql中进行查询，也可以查到隔离级别，如下所示：

![image-20220107225113087](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220208144003.png) 

相对其他数据库而言，MySQL的锁机制比较简单，其最显著的特点是不同的存储引擎支持不同的锁机制。下表是常见存储引擎对锁的支持情况： 

| 存储引擎 | 表级锁   | 行级锁   | 页面锁 |
| -------- | -------- | -------- | ------ |
| `MyISAM` | ==支持== | 不支持   | 不支持 |
| `InnoDB` | ==支持== | ==支持== | 不支持 |
| `MEMORY` | ==支持== | 不支持   | 不支持 |

MySQL这3种锁的特性可大致归纳如下 ：

| 锁类型   | 特点                                                         |
| -------- | ------------------------------------------------------------ |
| `表级锁` | 偏向MyISAM存储引擎，开销小，加锁快；不会出现死锁；锁定粒度大，发生锁冲突的概率最高，并发度最低。 |
| `行级锁` | 偏向InnoDB存储引擎，开销大，加锁慢；会出现死锁；锁定粒度最小，发生锁冲突的概率最低，并发度也最高。 |
| `页面锁` | 开销和加锁时间界于表锁和行锁之间，会出现死锁；锁定粒度界于表锁和行锁之间，并发度一般。 |

##### 5.4.3 InnoDB行锁

InnoDB与MyISAM的最大不同有两点：一是**支持事务**；二是采用了**行级锁**。

###### 5.4.3.1 行锁的类型

**InnoDB主要实现了以下两种类型的行锁：**

- `共享锁`(S)：又称为读锁，简称S锁，共享锁就是多个事务对于同一数据可以共享一把锁，都能访问到数据，但是只能读不能修改。
- `排它锁`(X)：又称为写锁，简称X锁，排他锁就是不能与其他锁并存，如一个事务获取了一个数据行的排它锁，其他事务就不能再获取该行的其它锁，包括共享锁和排它锁，获取排它锁的事务可以对该行数据进行读取和修改。

我们也可以通过以下语句显示地给记录增加共享锁或排它锁：

```sql
共享锁：select * from table_name where ... lock in share mode;
排他锁：select * from table_name where ... for update;
```

> **注意：**对于update、delete以及insert语句，InnoDB会自动给涉及到的数据集添加排它锁；但对于普通select语句，InnoDB不会加任何锁。另外，需要注意的是，如果不通过索引条件检索数据，那么InnoDB将对表中的所有记录加锁，实际效果就跟表锁是一样的啦。

###### 5.4.3.2 关于间隙锁

当我们用范围条件，而不是使用相等条件检索数据并请求共享锁或排它锁时，InnoDB会给符合条件的已有数据进行加锁， 对于键值在条件范围内但并不存在的记录，叫做"间隙(GAP)"，InnoDB也会对这个"间隙"加锁，这种锁机制就是所谓的 `间隙锁`(Next-Key锁)。

> **说明**：对于键值在条件范围内但并不存在的记录，会被加上间隙锁。如果当前事务并未提交，另一个事务也是无法对加上间隙锁的这行数据进行操作的。

###### 5.4.3.3 优化建议

InnoDB存储引擎由于实现了行级锁定，虽然在锁定机制的实现方面带来了性能损耗可能比表锁会更高一些，但是在整体并发处理能力方面要远远优于MyISAM的表锁。当系统并发量较高的时候，InnoDB的整体性能和MyISAM相比就会有比较明显的优势。但是，InnoDB的行级锁同样也有其脆弱的一面，当我们使用不当的时候，可能会让InnoDB的整体性能表现不仅不能比MyISAM高，甚至可能会更差。

**优化建议：**

- 尽可能让所有数据检索都能通过索引来完成，避免无索引行锁升级为表锁；
- 合理设计索引，尽量缩小锁的范围；
- 尽可能减少索引条件及索引范围，避免间隙锁；
- 尽量控制事务大小，减少锁定资源量和时间长度；
- 尽可使用低级别事务隔离(但是需要业务层面满足需求)。

#### 5.5 mysql调优

##### 5.5.1 慢查询日志

我们可以通过慢查询日志来定位查询比较慢的SQL，使用以下命令查看是否开启慢查询日志以及该日志的存放路径：

```sql
show variables like '%slow_query_log%';
```

![image-20220108163157661](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002600.png) 

如果并未开始慢查询日志，那我们可以通过以下命令进行开启：

```sql
# 设置成1是开启，设置成0是关闭
set global slow_query_log=1;
```

![image-20220108163822340](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002606.png) 

在MYSQL中默认会记录一个慢查询SQL的最低阈值时间，查询耗时超过这个时间的SQL才会被记录到慢查询日志中，我们可以通过以下命令查看这个默认时间：

```sql
show variables like '%long_query_time%';
```

![image-20220108164436664](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002612.png) 

我们可以使用如下命令修改这个默认时间：

```sql
# 我这边表的数据量不大，为了触发写慢日志文件，所以这里设置为20毫秒
set global long_query_time=0.02;
```

设置之后需要重新连接一下mysql才能生效，生效后的效果如下：

![image-20220108172012478](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002618.png) 

生效之后，执行一条查询语句，查询时间超过20毫秒就会被记录到慢查询日志文件中，内容如下所示：

![image-20220108173020489](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002623.png) 

##### 5.5.2 explain执行计划分析

我们可以使用explain对SQL语句进行分析，从而得知语句执行的一些信息，比如下面这样：

```sql
explain select * from student where id = 2;
```

![image-20220108180619657](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002630.png) 

**字段含义详解：**

- `id`：select查询的序列号，整数类型，表示查询中执行select子句或者是操作表的顺序；
- ==select_type==：
  - `SIMPLE`：表示简单的select查询，查询中不包含子查询或union查询；
  - `PRIMARY`：查询中若包含任何复杂的子部分，最外层查询为PRIMARY，也就是最后加载的就是PRIMARY；
  - `SUBQUERY`：在select或where列表中包含了子查询，就会被标记为SUBQUERY；
  - `DERIVED`：在from列表中包含的子查询会被标记为DERIVED(衍生)，MySQL会递归执行这些子查询，将结果放在临时表中；
  - `UNION`：若第二个select出现在union后，则被标记为UNION，若union包含在from子句的子查询中，那么外层的select将被标记为DERIVED；
  - `UNION RESULT`：从union表获取结果的select会被标记为UNION RESULT。
- `table`：用于显示SQL操作的是哪张表；
- `partitions`：值为NULL表示表未被分区；
- ==type==：表的连接类型，即MySQL在表中找到所需行的方式，性能由好到差的的连接类型如下所示：
  - `system`：system是const类型的特例，当查询的表只有一行的情况下，使用system；
  - `const`：当MySQL对查询某部分进行优化，并转换为一个常量时，是这种类型。就是表最多只有一个匹配行时，比如使用主键或者唯一键查询的时候；
  - `eq_ref`：主键或唯一索引的所有部分被连接使用，最多只会返回一条符合条件的记录。这可能是const类型之外最好的联接类型，简单的select查询不会出现这种类型；
  - `ref`：相比eq_ref，不适用唯一索引，而是使用普通索引或者唯一索引的部分前缀，索引要和某个值相比较，可能会找到多个符合条件的行；
  - `range`：只检索给定范围的行，使用一个索引来检索行，可以在key列中查看使用的索引，一般出现在where语句的条件中，如使用between、>、<、in等查询。这种索引的范围扫描比全表扫描要好，因为索引的开始点和结束点都固定，不用扫描全索引；
  - `index`：全索引扫描，index和ALL的区别：index只遍历索引树，通常比ALL快，因为索引文件通常比数据文件小。虽说index和ALL都是全表扫描，但是index是从索引中读取，ALL是从磁盘中读取；
  - `ALL`：全表扫描，意味着MySQL需要从头到尾去查找所需要的行，这种情况下需要增加索引来进行优化。
- `possible_keys`：表示MySQL能使用哪个索引在表中找到记录，查询涉及到的字段上若存在索引，则该索引将会被列出，但不一定被查询使用；
- `key`：这一列显示MySQL实际采用哪个索引对该表的访问；
- `key_len`：这一列显示了mysql在索引里使用的字节数，通过这个值可以估算出具体使用了索引中的哪些列；
- `ref`：这一列显示了在key列记录的索引中，表查找值所用到的列或常量，常见的有const(常量)，字段名等。一般是查询条件或关联条件中等号右边的值，如果是常量那么ref列值就是const，否则的话就是字段名；
- `rows`：这个是根据表统计信息及索引选用情况，大致估算出找到所需记录所要读取的行数，注意这个不是结果集的行数，该值越小越好；
- `filtered`：百分比值，表示存储引擎返回的数据经过滤后，剩下多少满足查询条件记录数量的比例；
- ==Extra==：这是一个额外信息列，该列包含mysql解决查询的详细信息，有以下几种情况：
  - `Using index`：只使用索引树中的信息而不需要进一步搜索读取实际的行来检索表中的信息，避免访问表的额外数据行，效率不错；
  - `Using filesort`：表明mysql会对数据使用一个外部的索引排序，而不是按照表内的索引顺序进行读取，数据较小时从内存排序，否则需要在磁盘完成排序，出现Using filesort说明sql语句需要进行优化；
  - `Using where`：使用where语句来处理结果，查询的列未被索引覆盖，查询效率低；
  - `Using temporary`：出现这种表示mysql需要创建一张临时表来处理查询，这种情况常见于排序(ordey by)和分组查询(group by)，出现Using temporary也是要考虑进行优化的，比如索引优化等；
  - `Using index condition`：查询的列不完全被索引覆盖，where条件中是一个前导的范围；
  - `Select tables optimized away`：使用某些聚合函数(比如：max、min)来访问存在索引的某个字段。

##### 5.5.3 最左匹配原则

最左匹配原则都是针对联合索引而言的，是否满足最左匹配原则决定了索引的生效情况。下面进行实战演示，首先创建一个表，表中包含一个联合索引，如下所示：

```sql
CREATE TABLE `student` (
  `id` int(11) NOT NULL AUTO_INCREMENT COMMENT '主键',
  `name` varchar(32) DEFAULT NULL COMMENT '姓名',
  `age` int(11) DEFAULT NULL COMMENT '年龄',
  `chinese` int(11) DEFAULT NULL COMMENT '语文',
  `math` int(11) DEFAULT NULL COMMENT '数学',
  `english` int(11) DEFAULT NULL COMMENT '英语',
  PRIMARY KEY (`id`),
  KEY `union_index` (`chinese`,`math`,`english`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;
```

然后再在表中添加几条数据，如下所示：

```sql
insert into `student` (`name`, `age`, `chinese`, `math`, `english`) values('Tom','18','85','78','92');

insert into `student` (`name`, `age`, `chinese`, `math`, `english`) values('Lucy','20','86','88','90');

insert into `student` (`name`, `age`, `chinese`, `math`, `english`) values('Mark','20','92','68','83');
```

**包含联合索引所有字段查询时：**

```sql
explain select * from student where chinese=86 and math=88 and english=90;
explain select * from student where math=88 and chinese=86 and english=90;
explain select * from student where english=90 and math=88 and chinese=86;
```

![image-20220108212116945](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002640.png) 

> **结论**：由以上截图可知，用到了索引，where子句后的条件顺序调换不影响查询结果，因为mysql中有查询优化器，会自动优化查询顺序。

**包含联合索引第一个字段查询时：**

```sql
explain select * from student where chinese=86;
explain select * from student where chinese=86 and math=88;
explain select * from student where chinese=86 and english=90;
```

![image-20220108213044458](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002651.png) 

> **结论**：由上图可知，用到了索引，说明只要包含了联合索引的第一个字段，其他索引字段都使用的等号的话，就会用到索引。不过第三条语句中的两个字段不是联合索引中的连续字段，所以sql语句最终其实只用到了第一个字段的索引。

**不包含联合索引第一个字段查询时：**

```sql
explain select * from student where math=88;
explain select * from student where english=90;
explain select * from student where math=88 and english=90;
```

![image-20220108213726946](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002657.png) 

> **结论**：由以上截图可知，where子句后的查询条件不包含联合索引的第一个字段时，都没有用到索引，都是全表扫描。

##### 5.5.4 索引失效的情况

为了演示方便，这里先创建一个表，并在表中插入几条数据，如下所示：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `card` int(11) NOT NULL,
  `username` varchar(32) DEFAULT NULL,
  `password` varchar(32) DEFAULT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `unique_card` (`card`),
  UNIQUE KEY `unique_name` (`username`)
) ENGINE=InnoDB DEFAULT CHARSET=utf8;

insert into `user`(`card`,`username`,`password`) values(111,'gsl','Abc12345');
insert into `user`(`card`,`username`,`password`) values(222,'cqq','1qaz@WSX');
insert into `user`(`card`,`username`,`password`) values(333,'123','1q2w3e4r');
```

由于建表语句中对**card**字段和**username**字段都增加了唯一索引，所以正常情况下查询，索引都是会生效的，如下所示：

```sql
select * from user where card=222;
select * from user where username='gsl';
```

![image-20220108233258554](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002703.png) 

下面详细介绍几种索引失效的情况。

1. **where语句中包含or时，可能会导致索引失效：**

   ![image-20220108234242173](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002721.png) 

   > **说明**：除非or关联的查询条件都是主键才会使用索引，否则都会进行全表扫描。

2. **like通配符可能会导致索引失效：**

   ![image-20220108234908339](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002730.png) 

   > **说明**：当通配符在前面的时候，索引失效，进行全表扫描，通配符在后面的时候，索引不会失效。

3. **在索引列上使用内置函数，一定会导致索引失效：**

   ![image-20220108235359392](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002737.png) 

4. **隐式类型转换导致的索引失效：**

   ![image-20220108235638552](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002742.png) 

   > **说明**：像上面字段类型是字符串类型，但是该字段查询条件的值并没有加引号，就会触发隐式类型转换，从而导致索引失效。

5.  **where语句中索引列使用了负向查询，可能会导致索引失效：**

   ![image-20220109000110600](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002748.png) 

   > **说明**：负向查询包括：NOT、!=、<>、!<、!>、NOT IN、NOT LIKE等。

6. **where条件中对索引列进行运算，一定会导致索引失效：**

   ![image-20220109000612143](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002753.png) 

   > **说明**：在等号右边做运算是不会导致索引失效的，但是在等号左边做运算就可能会导致索引失效。

7. **联合索引下，查询条件不满足最左匹配原则，可能会导致索引失效：**

   > **说明**：关于这一点，失效场景已经在前面**最左匹配原则**章节演示过了，这里就不再重复演示了。

8. **使用is null或is not null时，可能会导致索引失效：**

   ![image-20220109001508309](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002757.png) 

   > **说明**：通过本次测试结果看，使用**is not null**导致索引失效了，但是使用**is null**并没有导致索引失效。

9. **对索引字段排序，可能会导致索引失效：**

   ![image-20220109002207654](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220109002803.png) 

   > **说明**：对主键索引排序的话，还是会用到索引的，但是对别的索引列排序的话，就会导致索引失效，除非select后面跟的字段只有索引列字段。

##### 5.5.5 SQL的优化方案

SQL的优化包括但不限于以下方案：

1. 对于经常被用来进行查询的字段，可以为其加上索引来提高查询的效率；
2. 使用`explain`对sql的执行计划进行分析，观察是否进行了全表扫描，对于那些使用了索引的字段，查看是否存在索引失效的情况；
3. 最好不要给数据库留null，尽可能地使用not null填充数据库；
4. select子句中尽量避免使用*号，需要查询哪些字段就写上哪些字段即可；
5. 尽量使用连接查询(join)替代子查询，因为连接查询不用在内存中创建临时表，效率会高一些；
6. 对于联合索引而言，要遵循最左匹配原则，避免索引失效；
7. 应尽量避免在where子句中使用!=或<>等操作符，否则将不会使用索引而进行全表扫描；
8. 应尽量避免在where子句中使用or来连接条件，如果一个字段有索引，一个字段没有索引，也会导致索引失效而进行全表扫描；
9. 应尽量避免使用in和not in，否则也会导致全表扫描。

#### 5.6 什么是MVCC

多版本并发控制(`MVCC`)：读取数据时通过一种类似快照的方式将数据保存下来，这样读锁和写锁就不冲突了，不同的事务session会看到自己特定版本的数据、版本链。

MVCC只在**READ_COMMITTED(读已提交)**和**REPEATABLE_READ(可重复读)**两个隔离级别下工作。其他隔离级别和MVCC不兼容，因为**(READ_UNCOMMITTED)读未提交**总是读取最新的数据行，而不是符合当前事务版本的数据行，而**串行化**则会对所有读取的行都加锁。

### 6.Linux篇

#### 6.1 grep命令的用法

常用参数及案例如下：

- `-A<num>`：打印出紧随匹配的行之后的下文num行，比如grep -A2 "JAVA" test.log；

  ![image-20220113192618510](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222009.png) 

- `-B<num>`：打印出紧随匹配的行之前的上文num行，比如grep -B2 "JAVA" test.log；

  ![image-20220113192746617](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222021.png) 

- `-c`：打印出匹配行的总数，比如grep -c "java" test.log；

  ![image-20220113193129769](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222051.png) 

- `-C<num>`：打印出匹配行的上下文前后各num行，比如grep -C "JAVA" test.log；

  ![image-20220113193729658](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222109.png) 

- `-E`：可以通过该参数实现and、or的效果，比如grep -E "cmdb.*costTime" test.log；

  ![image-20220113200521750](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222115.png) 

  > **说明**：我们也可以使用`egrep`，它和使用`grep -E`的效果是一样的。

- `-F`：不使用正则表达式，按照字符串字面意思进行匹配，比如grep -F "bcd.*" test.log；

  ![image-20220113201542565](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222128.png) 

- `-i`：匹配时，忽略字符串的大小写，比如grep -i "java" test.log；

  ![image-20220113201932679](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222134.png) 

- `-l`：只显示匹配指定字符串的文件名，不显示具体匹配行的内容，比如grep -l "JAVA" *；

  ![image-20220113205356237](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222139.png) 

- `-n`：显示过滤内容在文件中的行号，比如grep -n "java" test.log；

  ![image-20220113203004056](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222147.png) 

- `-o`：只显示匹配字符串的内容，比如grep -o "JAVA" test.log；

  ![image-20220113203324845](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222153.png) 

- `-v`：显示不包含匹配内容的所有行，比如grep -v "aaa" abc.log；

  ![image-20220113203746617](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222158.png) 

- `-w`：匹配整词，即只显示全字符和的列，比如grep -w "aaa" abc.log；

  ![image-20220113204337789](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222203.png) 

- `-x`：匹配整行，比如grep -x "I love JAVA very much！" test.log；

  ![image-20220113204751111](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222208.png) 

另外，grep也支持`正则表达式`，比如==.==表示一个字符，==^==表示以什么开头，==$==表示以什么结尾，==[0-9a-zA-Z]*==表示匹配所有数字和大小写字母。案例如下所示：

```bash
grep "costTime:\[...\]ms" test.log
grep "costTime:\[[0-9]*\]ms" test.log
grep "engine\[[0-9A-Z]*\]" test.log
grep "engine\[[a-z]*\]" test.log
grep "engine\[[0-9]*\]" test.log
```

![image-20220113222655225](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222708.png) 

![image-20220113211458353](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220113222218.png) 

#### 6.2 awk命令的用法

##### 6.2.1 直接打印指定列内容

```bash
cat netstat.txt|awk '{print $1, $4, $6}'
```

![image-20220115203827227](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224527.png) 

##### 6.2.2 对齐打印的内容

通过上面的操作可以发现，虽然可以打印出指定列的内容，但是格式看上去比较乱，不利于观察，我们可以使用类似下面的命令进行左对齐或者右对齐操作，如下所示：

```bash
# 左对齐
cat netstat.txt|awk '{printf "%-10s %-15s %-s\n",$1, $4, $6}'

# 右对齐
cat netstat.txt|awk '{printf "%+10s %+15s %+10s\n",$1, $4, $6}'
```

![image-20220115212133295](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224535.png) 

> **说明**：如果打印的列是字符串的话，格式符就用`%s`，如果是十进制的整数，就用`%d`；左对齐就使用`-`，右对齐就使用`+`；数字代表多少空格，比如`%-10s`就表示列是字符串，使用的对齐方式是左对齐，并且和下一列间隔10个空格。

##### 6.2.3 根据指定字符拼接打印内容

```bash
cat netstat.txt|awk '{print $1 "|" $4 "|" $6}'
```

![image-20220115204246941](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224541.png) 

##### 6.2.4 根据条件打印指定列内容

```bash
cat netstat.txt|awk '$1=="tcp6" && $2==1 {print $0}'
```

![image-20220115205257787](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224549.png) 

> **说明**：上图中的`print $0`的意思是打印所有列的内容。

##### 6.2.5 展示指定行的内容

虽然我们可以根据条件打印指定列的内容，但是由于表头行并不满足条件要求，所以并没有打印出来。表头一般都是第一行的内容，所以如果我们希望打印出来的话，可以使用awk的内置变量`NR`。`NR`是用于获取指定行的内容的，从1开始。所以命令如下所示：

```bash
cat netstat.txt|awk '($1=="tcp6" && $2==1) || NR==1 {print $0}'
```

![image-20220115214607228](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224555.png) 

##### 6.2.6 改变默认分隔符打印内容

默认情况下使用awk，字段分隔符是空格或者Tab键，行分隔符是回车换行，下面就使用awk内置变量改变默认分隔符来演示效果，如下所示：

```bash
# 打印所有列
cat list.txt|awk 'BEGIN{RS=",";FS="|"} {print $0}'

# 打印指定列
cat list.txt|awk 'BEGIN{RS=",";FS="|"} {print $2, $3}'
```

![image-20220115221959099](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224601.png) 

> **说明**：上图中的`BEGIN`其实就是一个前置操作，比如当我们要使用`print`打印指定列时，我们可以将打印前需要执行的操作或者需要设置的内容放到`BEGIN`中。而`RS`则用于设置输入行的分隔符，默认是回车换行；`FS`则用于设置输入字段的分隔符，默认是空格或者Tab键。

##### 6.2.7 指定输入字段分隔符打印内容

```bash
# 使用"-F"参数的方式
cat test.txt|awk -F "|" '{print $2}'

# 使用内置变量"FS"的方式
cat test.txt|awk 'BEGIN{FS="|"} {print $2}'
```

![image-20220115224254594](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220115224644.png) 

#### 6.3 sed命令的用法

下面是为了方便演示而准备的素材，内容如下：

```xml
Str a = "The beautiful girl`s boy friend is Jack".
Str b = "Do you know Jack? I know, Jack is a real man".
Str c = "The beautiful girl loves Jack so mach".

This is test Str or STR.Please ignore!
Integer bf = new Integer(2);
```

##### 6.3.1 修改每行开头的内容

```bash
# 将每行开头的"Str"替换成"String"
cat replace.txt|sed 's/^Str/String/'
```

![image-20220116131135840](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152209.png) 

##### 6.3.2 修改每行结尾的内容

```bash
# 将每行结尾的"."替换成";"，由于"."是特殊字符，所以要转义
cat replace.txt|sed 's/\.$/;/'
```

![image-20220116134710831](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152225.png) 

##### 6.3.3 修改每行第一次出现的内容

```bash
# 将每行第一次出现的"Jack"替换成"me"
cat replace.txt|sed 's/Jack/me/'
```

![image-20220116135345452](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152232.png) 

##### 6.3.4 修改每行第二次出现的内容

```bash
# 将每行中的第二个"Jack"替换成"me"
cat replace.txt|sed 's/Jack/me/2g'
```

![image-20220116135815813](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152238.png) 

##### 6.3.5 修改每行指定的所有字符串

```bash
# 如下命令会将每行中所有包含"Jack"的内容都替换成"me"
cat replace.txt|sed 's/Jack/me/g'
```

![image-20220116140303650](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152244.png) 

> **说明**：使用`i`的话，在替换时可以忽略大小写，比如`cat replace.txt|sed 's/Jack/me/ig'`。这样一来，即便某些行中存在"JACK"、"jaCK"这种字符串，也都会被替换成"me"。

##### 6.3.6 删除指定行的内容

```bash
# 删除空行和以"Integer"开头的行
cat replace.txt|sed '/^ *$/d;/^Integer/d'
```

![image-20220116140744057](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152250.png) 

##### 6.3.7 在行前或行后增加指定内容

```bash
# 在包含指定字符串的行的行前增加指定内容
cat replace.txt|sed '/beautiful/i 12345abcde'

# 在包含指定字符串的行的行后增加指定内容
cat replace.txt|sed '/beautiful/a abcde12345'
```

![image-20220116143235443](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152259.png) 

![image-20220116143545947](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152306.png) 

##### 6.3.8 修改原文件的内容

以上演示的内容都是通过管道符对输出流的展示做操作，并不会修改原文件的内容，如果我们希望直接对原文件进行修改的话，可以使用`-i`参数，举例如下：

```bash
sed -i '/^ *$/d;/^Integer/d' replace.txt
```

![image-20220116145315276](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152315.png) 

#### 6.4 cut命令和sort命令

为了演示方便，现准备如下素材：

```xml
1,Chinese,85
2,English,76
3,Math,92
```

我们可以使用`cut`命令指定分隔符进行截取，并展示指定列的内容；对于展示的指定列，如果符合自然排序规则，我们可以使用`sort`命令进行排序，演示如下：

```bash
# 使用逗号作为分隔符，分割后展示第二列、第三列的内容
cat score.txt|cut -d "," -f 2,3

# 对展示的指定列进行正序排列
cat score.txt|cut -d "," -f 3|sort -n

# 对展示的指定列进行倒序排列
cat score.txt|cut -d "," -f 3|sort -nr
```

![image-20220116152112915](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220116152322.png) 

### 7.Redis篇

#### 7.1 Redis简介

Redis是一个使用C语言开发的数据库，与传统数据库不同的是，Redis的数据是存在内存中的，属于内存数据库，所以读写速度非常快，因此Redis被广泛应用于缓存方向。

另外，Redis除了做缓存之外，也经常用来做分布式锁，甚至是消息队列。Redis提供了多种数据类型来支持不同的业务场景。Redis还支持事务、持久化、Lua脚本、多种集群方案。

#### 7.2 Redis的数据结构及使用场景

Redis的数据结构及使用场景如下：

1. `String(字符串)`：常用的命令有set、get、decr、incr、mget等。字符串是最常用的数据类型，它可以用来存储某个简单的字符串，也可以是二进制、json化的对象，甚至是Base64编码后的图片，在redis中，一个字符串最大的容量是512MB。我们经常使用的缓存功能就可以使用这种数据类型，这种数据类型还可以用作Redis分布式锁、计数器、Session共享以及分布式ID等。
2. `Hash(哈希)`：常用的命令有hget、hset、hgetall、hdel等。这种数据结构可以用来存储一些key-value的键值对，特别适合用于存储对象。比如，我们可以使用hash数据结构来存储用户信息、商品信息等。
3. `List(列表)`：常用的命令有lpush、rpush、lpop、rpop、lrange等。这是一种双向链表结构，可以支持反向查找和遍历，更方便操作，不过会带来部分额外的内存开销。它既可以当做栈，也可以当做队列来使用。可以用来缓存类似微信公众号、微博等消息流数据。
4. `Set(集合)`：常用的命令有sadd、spop、smembers、sunion等。和List这种数据类型类似，都可以存储多个元素，但是使用Set的话，元素是不能重复的，所以这种数据类型能够轻易实现交集、并集、差集的操作，更适用于实现如共同关注、共同粉丝、共同喜好等功能。
5. `Sorted Set(有序集合)`：常用的命令有zadd、zrange、zrem、zcard等。Set是无序的，而Sorted Set增加了一个权重参数score，使得集合中的元素能够按score进行有序排列，所以有序集合可以用来实现排行榜等功能。

#### 7.3 Redis快的原因

众所周知，Redis拥有10W+的QPS，Redis之所以这么快，主要有以下几个原因：

- 完全基于内存，绝大部分请求都是纯粹的内存操作，执行效率高；
- 数据结构简单，对数据操作也简单；
- 在处理网络请求时采用单线程的模型，保证了每个操作的原子性，也避免了因为线程的上下文切换和竞争而导致的性能消耗。但是单线程的方式可能无法发挥多核CPU性能，所以想多核可以启动多个redis实例；
- 使用多路I/O复用模型，非阻塞IO。"多路"指的是多个网络连接，"复用"指的是复用同一个线程，采用多路I/O复用技术可以让单个线程高效地处理多个连接请求，所以redis的单线程也能处理高并发请求。

#### 7.4 Redis的多线程设计

在Redis6.0之前的版本中，都还是使用的单线程设计，主要是由于以下原因：

- 单线程编程容易并且更易于维护；
- Redis的性能瓶颈不在CPU，主要在内存和网络；
- 多线程容易存在死锁、线程上下文切换等问题，会影响性能。

之所以在Redis6.0版本之后又引入了多线程，可能有以下原因：

- Redis的性能瓶颈主要在内存和网络，使用多线程可以提高网络IO的读写性能；
- 使用多线程可以充分利用服务器的CPU资源，使用多线程可以充分利用多核；
- 虽然Redis6.0引入了多线程，但是Redis的多线程只是在网络数据的读写这类耗时操作上使用了，执行命令仍然是单线程顺序执行。因此，我们仍然不需要担心线程安全问题。

> **说明**：Redis6.0的多线程默认是禁用的，如需开启需要修改redis.conf文件的配置，开启多线程后，还需要通过修改redis.conf文件来设置线程数，否则是不生效的。

#### 7.5 缓存穿透、击穿和雪崩

##### 7.5.1 缓存的穿透

**缓存穿透**就是大量请求的key根本不存在于缓存中，导致请求直接打到了数据库上，根本没有经过缓存这一层。比如某个黑客故意制造我们缓存中不存在的key发起大量请求，导致大量请求落到数据库。

对于缓存穿透的情况，业界一般有两种解决办法：

- `缓存空对象`：如果在缓存和数据库中都查不到某个key的数据，那么就给该key在缓存中设置一个空对象，并设置过期时间。这种方式可以解决请求的key变化不频繁的情况，如果黑客恶意攻击，每次构建不同的请求key，就会导致缓存中存在大量无效的key。很明显，这种方案并不能从根本上解决此问题。如果非要用这种方式来解决穿透问题的话，尽量将无效key的过期时间设置的短一点。
- `布隆过滤器拦截`：把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话请求才会再进入缓存以及数据库中，可以使用bitmap做布隆过滤器。但是这种方式可能会存在误判的情况。

##### 7.5.2 缓存的击穿

对于设置了过期时间的key在某些时间点被超高并发访问，说明它是一种非常"热点"的数据。如果这个key在大量请求同时进来前正好失效了，那么所有针对这个key的操作都会落到数据库上，就会出现持久层负载过大甚至奔溃的情况，这种情况被称为**缓存击穿**。

对于缓存击穿的情况，业界一般有两种解决办法：

- `设置value永不过期`：这种方式可以说是最可靠、最安全的，但是占空间，内存消耗大，并且不能保持数据最新，所以这种方案需要考虑到具体的业务逻辑。
- `使用互斥锁(mutex key)`：这种方式的实现思路就是，在缓存失效的时候(即根据key获取的值为空)不是立即去查数据库，而是先使用缓存的类似setnx命令去set一个mutex key，设置成功再查询数据库，然后将查询的结果设置到缓存中去。这里的setnx就起到了互斥锁的作用，相当于大量并发请求只让一个请求去查数据库，其他请求等待。查到后将数据设置到缓存中并释放锁即可，这样其他请求就可以从缓存中获取到数据了，就不用再去查数据库了。

##### 7.5.3 缓存的雪崩

**缓存雪崩**是指缓存层由于某些原因不可用(比如宕机)，或者一些被大量请求访问的数据的key在同一时间大面积失效，导致对应的请求直接落到了数据库上，造成数据库短时间内承受大量请求压力。

对于缓存雪崩的情况，可以采用如下解决手段：

- 如果是redis服务不可用导致的雪崩情况，可以采用redis集群方式解决，避免由于单机出现问题导致整个缓存服务都没办法使用的情况；
- 如果是由于缓存大面积失效导致的雪崩，可以使用随机数作为缓存的过期时间，以避免大量key同时过期的情况。

#### 7.6 Redis的持久化

##### 7.6.1 RDB持久化

RDB全称是Redis Database Backup file(Redis数据备份文件)，也被叫做Redis数据快照。简单来说就是把内存中的所有数据都记录到磁盘中。当Redis实例故障重启后，从磁盘读取快照文件，恢复数据。快照文件称为RDB文件，默认是保存在当前运行目录。

Redis停机时会执行一次RDB。我们也可以通过`save`或者`bgsave`命令执行RDB，区别在于，`save`命令是由redis主进程来执行RDB，而redis是单线程设计，所以这时会阻塞所有命令；而`bgsave`命令则是由一个异步的子进程来执行RDB，所以并不会阻塞主进程。

Redis内部有触发RDB的机制，可以在redis.conf文件中找到，格式如下：

![image-20220118213451227](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222602.png)  

RDB的其它配置也可以在redis.conf文件中设置，例如：

![image-20220118213519188](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222607.png) 

bgsave底层在工作时，开始会fork主进程得到子进程，子进程共享主进程的内存数据，完成fork后会读取内存数据并写入RDB文件。

fork采用的是copy-on-write技术：

- 当主进程执行读操作时，访问共享内存；
- 当主进程执行写操作时，则会拷贝一份数据，执行写操作。

下图是bgsave底层执行流程的原理图：

![image-20220118214059348](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222611.png) 

**RDB的优缺点：**

- **优点：**
  - 对于灾难恢复而言，RDB是非常不错的选择；
  - redis加载RDB文件恢复数据远远快于AOF方式；
  - RDB是一个快照文件，数据很紧凑，它保存了redis在某个时间点上的数据集，体积比较小。
- **缺点：**
  - RDB执行间隔时间长，两次RDB之间写入数据有丢失的风险；
  - RDB是通过fork子进程来协助完成数据持久化工作的，当数据集较大时，fork子进程、压缩、写出RDB文件这一系列流程是比较耗时的。

##### 7.6.2 AOF持久化

AOF全称为Append Only File(追加文件)。redis处理的每一个写命令都会记录在AOF文件，所以AOF文件其实是记录命令的，因此可以看做是一个命令日志文件。

AOF默认是关闭的，需要修改redis.conf配置文件来开启AOF，如下所示：

![image-20220118220521665](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222753.png) 

AOF的命令记录频率也可以通过redis.conf文件来修改，如下所示：

![image-20220118220849837](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222758.png) 

> **说明**：一般情况下，AOF的命令记录频率是不建议修改的，就使用默认方案即可。

由于是记录命令，所以AOF文件会比RDB文件大的多。而且AOF会记录对同一个key的多次写操作，但其实只有最后一次写操作才有意义。通过执行`bgrewriteaof`命令，可以让AOF文件执行重写功能，即用最少的命令达到相同效果，如下所示：

![image-20220118221429060](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222805.png) 

redis也会在触发阈值时自动去重写AOF文件，阈值也可以在redis.conf中配置：

![image-20220118221545056](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222815.png) 

AOF和RDB是各有优缺点的，它们的主要区别如下：

![image-20220118221850519](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220118222824.png) 

> **说明**：使用AOF方式最大的好处就是能够较好地保证数据的完整性，其实在实际场景中，RDB和AOF这两种方式一般是会被结合起来使用的。

#### 7.7 Redis的集群搭建

Redis集群的搭建主要包含三种方式，分别是`Redis主从集群`、`Redis哨兵集群`、`Redis分片集群`。

##### 7.7.1 Redis主从集群

###### 7.7.1.1 主从集群架构

Redis的主从集群基础架构如下所示：

![image-20220120224817137](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220122225111.png) 

我们可以在某个Redis节点上通过`slaveof ip port`命令将该节点加入到Redis集群中，加入集群后该节点会成为该ip和port对应的Redis的slave节点。

由于主从之间需要进行数据同步，所以如果从节点过多的话，会大大增加主节点的压力，这时候就可以使用主-从-从的链式结构，从而减小主节点的压力，此时的集群架构如下所示：

![image-20220122204540810](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220122225119.png) 

###### 7.7.1.2 主从数据同步原理

主从节点之间的数据同步主要包括`全量同步`和`增量同步`。主从第一次同步就是全量同步，流程如下所示：

![image-20220122210518926](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220122225125.png) 

master如何判断slave是不是第一次来同步数据，这里主要是用到两个很重要的概念：

- `Replication Id`：简称replid，是数据集的标记，id一致则说明是同一数据集，每个master都有唯一的replid，slave则会继承master节点的replid；
- `offset`：偏移量，随着记录在repl_baklog中的数据增多而逐渐增大。当slave在完成同步时，也会记录当前同步的offset。如果slave的offset小于master的offset，说明slave数据落后于master，则需要更新。

**因此，slave做数据同步，必须向master声明自己的replid和offset，master才可以判断到底需要同步哪些数据。**

全量同步的基本流程如下：

1. slave节点向master节点请求增量同步；
2. master节点判断replid，发现不一致，拒绝增量同步；
3. master将完整内存数据生成RDB，发送RDB到slave，开始全量同步；
4. slave清空本地数据，加载master的RDB数据；
5. master将RDB期间的命令记录在repl_baklog，并持续将log中的命令发送给slave；
6. slave执行接收到的命令，以保持与master之间的同步。

主从第一次同步是全量同步，但如果slave重启后再同步，则是**增量同步**，流程如下所示：

![image-20220122224121495](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220122225137.png) 

**Redis主从集群的优化手段：**

- 在master的redis.conf配置文件中配置repl-diskless-sync属性值为yes来启用无磁盘复制，避免全量同步时的磁盘IO(网络带宽较好时可采用此方案)；
- Redis单节点上的内存占用不要太大，减少RDB导致的过多磁盘IO；
- 适当提高repl_baklog的大小，发现slave宕机时尽快实现故障恢复，尽可能避免全量同步；
- 限制一个master上的slave节点数量，如果实在有太多slave，则可以采用主-从-从链式结构，减少master压力。

**总结：**

**何时执行全量同步：**

- slave节点第一次连接master节点时；
- slave节点断开时间太久，repl_baklog中的offset已经被覆盖时。

**何时执行增量同步：**

- slave节点断开又恢复，并且在repl_baklog中能找到offset时。

##### 7.7.2 Redis哨兵集群

###### 7.7.2.1 哨兵的结构和作用

Redis提供了哨兵(Sentinel)机制来实现主从集群的自动故障恢复。哨兵的结构和作用如下：

- `监控`：Sentinel会不断检查我们的master和slave是否按预期工作；
- `自动故障恢复`：如果master出现了故障，Sentinel会将一个slave提升为master，故障的master恢复后将会作为slave运行；
- `通知`：Sentinel充当Redis客户端的服务发现来源，当集群发生故障转移时，会将最新信息推送给Redis的客户端。

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123211211.png" alt="image-20220123200832264" style="zoom:80%;" /> 

> **说明**：Redis的哨兵机制本身也是一个集群，不然如果自己本身出现了故障，就无法起到监测集群的作用了。

###### 7.7.2.2 哨兵的服务状态监测

Sentinel哨兵基于心跳机制监测服务状态，每隔1秒会向集群的每个实例发送ping命令：

- `主观下线`：如果某sentinel节点发现某实例未在规定时间响应，则认为该实例**主观下线**；
- `客观下线`：若超过指定数量(quorum)的sentinel都认为该实例主观下线，则该实例**客观下线**。quorum的值最好超过Sentinel实例数量的一半。

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123211219.png" alt="image-20220123201737449" style="zoom:85%;" /> 

> **说明**：如上图所示，当三个Sentinel实例中有两个都认为master实例主观下线，则该master实例客观下线，这时就会选举新的master节点。

###### 7.7.2.3 master的选举依据

一旦发现master故障，sentinel就需要在salve中选择一个作为新的master，选举的依据如下：

1. 首先会判断slave节点与master节点断开时间长短，如果超过指定值(down-after-milliseconds * 10)，数据的完整性得不到保证，则会排除该slave节点；
2. 然后会判断slave节点的slave-priority值，越小优先级越高，默认都是一样的，如果是0则永不参与选举；
3. 如果slave-prority值一样，则判断slave节点的offset值，越大说明数据越新，优先级越高；
4. 如果offset值也一样，说明选哪个slave都一样，那么就会随机选择一个slave。这里在随机选择时，是根据slave节点的运行id大小做判断的，越小优先级越高。

###### 7.7.2.4 实现故障转移的步骤

当选中了其中一个slave为新的master后，实现故障转移的步骤如下：

- Sentinel会给备选的slave节点发送`slaveof no one`命令，让该节点成为master节点；
- 然后会给所有其他的slave节点发送`slaveof 新master的ip 新master的端口`命令，让这些slave成为新master的从节点，并开始从新的master节点上同步数据；
- 最后，Sentinel会将故障节点标记为slave节点，当故障节点恢复后会自动成为新master节点的slave节点。

###### 7.7.2.5 RedisTemplate的哨兵模式

在Sentinel集群监管下的Redis主从集群，其节点会因为自动故障转移而发生变化，Redis的客户端必须要能感知到这种变化，并及时更新连接信息。Spring的RedisTemplate底层利用lettuce实现了节点的感知和自动切换。

在application.yml配置文件中，可以通过如下方式进行哨兵的配置：

```yaml
spring:
  redis:
    sentinel:
      master: mymaster  # 指定master名称
      nodes:  # 指定redis-sentinel的集群信息
        - 192.168.68.10:7001
        - 192.168.68.11:7001
        - 192.168.68.12:7001
```

> **说明**：以上配置的集群信息是redis哨兵集群的集群信息，并不是redis节点的集群信息。

主从读写分离策略的配置：

```java
@Bean
public LettuceClientConfigurationBuilderCustomizer configurationBuilderCustomizer() {
    return configBuilder -> configBuilder.readFrom(ReadFrom.REPLICA_PREFERRED);
}
```

以上的ReadFrom用于配置Redis的读取策略，是一个枚举，包括下面这些选项：

- `MASTER`：从主节点读取；
- `MASTER_PREFERRED`：优先从master节点读取，master不可用则从slave节点读取；
- `REPLICA`：从slave节点读取；
- `REPLICA_PREFERRED`：优先从slave节点读取，所有的slave都不可用再从master节点读取。

> **说明**：Redis集群一般都是读写分离，master节点用于接收写操作，slave节点则用于接收读操作。所以在配置读取策略时，一般都是优先去slave节点读取数据以减轻master节点的压力。

##### 7.7.3 Redis分片集群

使用Redis的分片集群将不再需要哨兵机制，因为其本身已经具备了哨兵机制的所有功能。

###### 7.7.3.1 分片集群的结构

主从集群和哨兵集群可以解决高可用、高并发读的问题。但是依然会存在以下两个问题：

- 海量数据存储问题；
- 高并发写的问题。

使用分片集群则可以解决上述问题，分片集群的特征如下：

- 集群中有多个master，每个master保存不同的数据；
- 每个master都可以有多个slave节点；
- 每个master之间通过ping监测彼此的健康状态；
- 客户端请求可以访问集群的任意节点，最终都会被转发到正确的节点上。

<img src="https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123223212.png" alt="image-20220123214657462" style="zoom:84%;" /> 

###### 7.7.3.2 分片集群的散列插槽

Redis会把每一个master节点映射到0~16383共**16384**个插槽(hash slot)上，查看集群信息时就能看到，比如：

![image-20220123215141619](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123223224.png) 

存到redis中的数据的key并不是与节点绑定，而是与插槽绑定。redis会根据key的有效部分计算插槽值，分两种情况：

- key中包含"{}"，且"{}"中至少包含1个字符，"{}"中的部分是有效部分；
- 如果key中不包含"{}"，则整个key都是有效部分。

> **说明**：计算插槽的方式是根据key的有效部分通过CRC16算法先得到一个hash值，然后对16384取余，余数将会被作为插槽。每个redis节点都会有一个插槽范围，插槽在这个范围内，对应的key就会被存到这个实例上。

###### 7.7.3.3 分片集群的故障转移

在redis的分片集群中，也是可以实现自动故障转移的，如果有一个master节点宕机了，会有如下现象：

1. 首先是该实例与其它实例会失去连接；

2. 然后是疑似宕机，如下所示：

   ![image-20220123221010432](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123223233.png) 

3. 最后是确定下线，并自动提升一个slave为新的master节点，如下所示：

   ![image-20220123221110948](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123223253.png) 

我们也可以在slave节点上使用`cluster failover`命令手动让集群中的某个master宕机，这时候执行该命令的slave节点就会变成master节点，宕机的master节点就会变成slave节点，并实现无感知的数据迁移。其流程如下：

![image-20220123221916130](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220123223352.png) 

手动的Failover支持三种不同的模式：

- `缺省`：这个是默认的模式；
- `force`：这种模式省略了对offset的一致性校验；
- `takeover`：相当于直接执行上图的第5步，忽略数据一致性、忽略master状态和其它master的意见。

###### 7.7.3.4 RedisTemplate访问分片集群

RedisTemplate底层同样基于lettuce实现了分片集群的支持，使用的步骤与哨兵模式基本一致。与哨兵模式相比，其中只有分片集群的配置方式略有差异，分片集群在application.yml中的配置如下：

```yaml
spring:
  redis:
    cluster:
      nodes:  # 指定分片集群中每一个节点的信息
        - 192.168.68.10:8001
        - 192.168.68.10:8002
        - 192.168.68.10:8003
        - 192.168.68.11:8001
        - 192.168.68.11:8002
        - 192.168.68.11:8003
```

> **说明**：分片集群中的每一个节点都是实际使用的redis的节点，所以在配置的时候直接配置redis的节点信息即可，而不是像哨兵集群那样配置哨兵集群的节点信息。

#### 7.8 Redis的分布式锁

像`synchronized`这种本地锁对于单体实例是有效果的，但是对于分布式场景，服务是会进行拆分的，而且我们的应用实例也会有多个，这时再使用本地锁就无法锁住整个服务了，顶多就是锁住服务的一个实例而已，所以就出现了分布式锁，而使用redis就可以实现分布式锁的效果。

使用redis作为分布式锁主要借助于setnx命令，伪代码如下所示：

```java
package com.gongsl.test;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.core.script.DefaultRedisScript;
import org.springframework.stereotype.Component;
import java.util.Arrays;
import java.util.UUID;
import java.util.concurrent.TimeUnit;

/**
 * @Author: gongsl
 * @Date: 2022-01-24 16:30
 */
@Component
public class TestLock {

    public static final String LOCK_KEY = "lock";

    @Autowired
    private StringRedisTemplate redisTemplate;

    public void getLock(){
        //使用uuid作为value值是为了防止并发场景下删除锁时的误删问题，判断uuid一致后再删，就能保证删除的是还没过期的自身设置的锁
        String token = UUID.randomUUID().toString();
        Boolean lock = redisTemplate.opsForValue().setIfAbsent(LOCK_KEY, token, 300, TimeUnit.SECONDS);
        if (lock) {
            try {
                //这里是业务逻辑
            }finally {
                //uuid判断成功后再删的逻辑也不是原子性的，也可能出现问题，所以这里使用Lua脚本的方式进行判断并删除锁
                String script = "if redis.call('get',KEYS[1]) == ARGV[1] then return redis.call('del',KEYS[1]) else return 0 end";
                redisTemplate.execute(new DefaultRedisScript<Integer>(script, Integer.class), Arrays.asList(LOCK_KEY), token);
            }
        }
    }
}
```

![image-20220124165917977](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220124193743.png) 

> **说明**：以上代码中的Lua脚本是直接从redis官网粘贴过来的，我们也可以直接到[官网](https://redis.io/commands/set)中进行查看。其实针对redis的分布式锁有一个专门的[Redisson](https://github.com/redisson/redisson/wiki/8.-%E5%88%86%E5%B8%83%E5%BC%8F%E9%94%81%E5%92%8C%E5%90%8C%E6%AD%A5%E5%99%A8)框架，我们也可以通过这个框架实现分布式锁的效果。

#### 7.9 缓存过期策略和淘汰算法

##### 7.9.1 缓存过期策略

缓存过期策略一般有以下三种：

- `定时过期`：每个设置过期时间的key都需要创建一个定时器，到过期时间就会立即清除。该策略可以立即清除过期的数据，对内存很友好；但是会占用大量的CPU资源去处理过期的数据，从而影响缓存的响应时间和吞吐量。
- `惰性过期`：只有当访问一个key时，才会判断该key是否已过期，过期则清除。该策略可以最大化地节省CPU资源，却对内存非常不友好。极端情况可能出现大量的过期key没有再次被访问，从而不会被清除，占用大量内存。
- `定期过期`：每隔一定的时间，会扫描一定数量的数据库的expires字典中一定数量的key，并清除其中已过期的key。该策略是前两者的一个折中方案。通过调整定时扫描的时间间隔和每次扫描的限定耗时，可以在不同情况下使得CPU和内存资源达到最优的平衡效果。

> **说明**：Redis中同时使用了`惰性过期`和`定期过期`两种过期策略。**定时过期**策略用一个定时器来负责监视key，过期则自动删除，这种方式比较消耗CPU资源，因此Redis没有采用这一策略。

##### 7.9.2 常见的缓存淘汰算法

常见的缓存淘汰算法主要有以下三种：

- `FIFO`(First In First Out，先进先出)：根据缓存被存储的时间，离当前最远的数据优先被淘汰；
- `LRU`(Least Recently Used，最近最少使用)：根据最近被使用的时间，离当前最远的数据优先被淘汰；
- `LFU`(Least Frequently Used，最不经常使用)：在一段时间内，缓存数据被使用次数最少的会被淘汰。

> **说明**：Redis中使用了`LRU`和`LFU`两种淘汰算法。

### 9.网络协议篇

#### 9.1 TCP和UDP的主要区别

- TCP是面向连接的，UDP是无连接的；
- TCP是面向字节流的，UDP是面向报文的；
- TCP对系统资源要求较多，UDP对系统资源要求较少；
- UDP具有较好的实时性，工作效率比TCP高；
- TCP可以提供可靠的服务，UDP是不可靠的，可能出现丢包的情况；
- TCP一般用于文件传输、发送和接收邮件、远程登录等场景，UDP一般用于即时通信，比如QQ语音、直播等。

#### 9.2 TCP为什么要进行三次握手

三次握手的目的是建立可靠的通信信道，主要就是双方为了确认自己与对方的发送与接收是正常的。

第一次握手：Client什么都不能确认；Server确认了对方发送正常，自己接收正常；
第二次握手：Client确认了：自己发送、接收正常，对方发送、接收正常；Server确认了：对方发送正常，自己接收正常；
第三次握手：Client确认了：自己发送、接收正常，对方发送、接收正常；Server确认了：自己发送、接收正常，对方发送、接收正常。
所以三次握手就能确认双方收发功能都正常，缺一不可。

#### 9.3 断开一个TCP连接为什么要进行四次挥手

- 客户端-发送一个FIN，用来关闭客户端到服务器的数据传送；
- 服务器-收到这个FIN，它发回一个ACK，确认序号为收到的序号加1。和SYN一样，一个FIN将占用一个序号；
- 服务器-关闭与客户端的连接，发送一个FIN给客户端；
- 客户端-发回ACK报文确认，并将确认序号设置为收到序号加1。

任何一方都可以在数据传送结束后发出连接释放的通知，待对方确认后进入半关闭状态。当另一方也没有数据再发送的时候，则发出连接释放通知，对方确认后就完全关闭了TCP连接。

#### 9.4 HTTP和HTTPS的区别

- https协议需要到CA申请证书，一般免费证书较少，因而需要一定的费用，而http则不需要；
- http是超文本传输协议，信息是明文传输，https则是具有安全性的密文传输；
- http和https使用的是完全不同的连接方式，默认使用的端口也不一样，http默认使用80端口，而https默认使用443端口。

#### 9.5 URI和URL的区别

- URI(Uniform Resource Identifier)是统一资源标志符，可以唯一标识一个资源；
- URL(Uniform Resource Locator)是统一资源定位符，可以提供该资源的路径。它是一种具体的URI，URL不仅可以用来标识一个资源，而且还指明了如何定位这个资源；
- URI的作用像身份证号一样，URL的作用更像家庭住址一样，URL是一种具体的URI。

#### 9.6 对称加密和非对称加密的区别

- 对称密钥加密是指加密和解密使用同一个密钥的方式，这种方式存在的最大问题就是密钥发送问题，即如何安全地将密钥发给对方；
- 非对称加密是指使用一对非对称密钥，即公钥和私钥，公钥可以随意发布，但私钥只有自己知道。送密文的一方使用对方的公钥进行加密处理，对方接收到加密信息后，使用自己的私钥进行解密；
- 由于非对称加密的方式不需要发送用来解密的私钥，所以可以保证安全性；但是和对称加密比起来，它较慢，所以我们可以用对称加密来传送消息，而对称加密所使用的密钥可以通过非对称加密的方式发送出去。

#### 9.7 Cookie和Session的区别

- Cookie一般用来保存用户信息，Session的主要作用就是通过服务端记录用户的状态；
- Cookie数据保存在客户端(浏览器端)，Session数据保存在服务器端，所以相对来说Session安全性更高；
- 由于Session数据是存放在服务器端上的，所以当访问增多时会占用服务器的性能。

#### 9.8 GET请求与POST请求的区别

- 使用GET方式提交，参数是跟在地址栏后面的，而使用POST方式提交，参数是跟在请求的实体内容中的；
- POST方式提交更加安全，所以可以用来传递敏感信息；
- 使用GET方式提交的话，在地址栏中传送的参数是有长度限制的，而使用POST方式提交则没有限制。

#### 9.9 HTTP和RPC的区别

- HTTP是协议，RPC是一种概念、一种设计，是为了解决不同服务之间的调用问题的，它一般会包含**传输协议**和**序列化协议**这两个；
- RPC的实现可以基于TCP协议，也可以基于HTTP协议；
- 使用RPC可以实现高效的二进制传输，HTTP大部分是通过JSON实现的，字节大小和序列化耗时都比RPC更消耗性能；
- RPC主要用于服务的远程调用，并且性能消耗低，传输效率高，服务治理方便；
- 良好的RPC调用是面向服务的封装，针对服务的可用性和效率等都做了优化，单纯使用HTTP调用则缺少了这些特性。

### 10.算法篇

#### 10.1 二分查找法

相关代码如下所示：

```java
package com.gongsl.test.array;

/**
 * @Author: gongsl
 * @Date: 2022-01-29 21:19
 * @Describe: 二分查找法
 */
public class BinarySearch {
    public static void main(String[] args) {
        //必须是排好序的数组才能使用二分查找法
        int[] array = {1, 5, 8, 21, 35, 46, 52, 76, 81};
        int target = 52;
        int index = binarySearch(array, target);
        System.out.println("index = " + index);
    }

    //二分查找，找到返回元素索引，找不到返回-1
    public static int binarySearch(int[] arr, int target) {
        //left：左边界，right：右边界
        int left = 0, right = arr.length - 1, mid;
        while (left <= right) {
            //left + right有可能超过整数最大值导致整数溢出，所以这里使用无符号右移的方式
            mid = (left + right) >>> 1;
            if (arr[mid] == target) {
                return mid;
            } else if (arr[mid] > target) {
                right = mid - 1;
            } else {
                left = mid + 1;
            }
        }
        return -1;
    }
}

```

> **说明**：这里的二分查找法参考的是`java.util.Arrays#binarySearch`方法中的写法。

#### 10.2 冒泡排序法

##### 10.2.1 演示

冒泡排序的效果演示[动图](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220221155312.gif)如下所示：

![冒泡排序](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220221155312.gif) 

##### 10.2.2 基础版

相关代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-03 18:54
 */
public class BubbleSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        bubbleSort(array);
    }

    public static void bubbleSort(int[] arr) {
        for (int j = 0; j < arr.length - 1; j++) {
            System.out.print("每轮比较次数：");
            for (int i = 0; i < arr.length - 1 - j; i++) {
                System.out.print((i + 1) + " ");
                if (arr[i] > arr[i + 1]) {
                    int temp = arr[i];
                    arr[i] = arr[i + 1];
                    arr[i + 1] = temp;
                }
            }
            System.out.println("第" + (j + 1) + "轮冒泡:" + Arrays.toString(arr));
        }
    }
}
```

以上代码的运行结果如下：

```java
每轮比较次数：1 2 3 4 5 6 7 第1轮冒泡:[5, 3, 4, 2, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 5 6 第2轮冒泡:[3, 4, 2, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 5 第3轮冒泡:[3, 2, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 第4轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3 第5轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 2 第6轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 第7轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
```

> **说明**：通过以上运行结果可知，里面的for循环使用`i<arr.length-1-j`条件后，避免了每轮的重复比较，所以每轮的比较次数是逐渐减少的。但是从运行结果看，在第4轮的时候就已经完成了排序，按理说只要排序完成，循环就应该结束的，所以以上代码还需要进行优化。

##### 10.2.3 优化版

优化后的代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-03 18:54
 */
public class BubbleSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        bubbleSort(array);
    }

    public static void bubbleSort(int[] arr) {
        for (int j = 0; j < arr.length - 1; j++) {
            System.out.print("每轮比较次数：");
            boolean swapped = false; //用于判断是否发生了位置交换
            for (int i = 0; i < arr.length - 1 - j; i++) {
                System.out.print((i + 1) + " ");
                if (arr[i] > arr[i + 1]) {
                    int temp = arr[i];
                    arr[i] = arr[i + 1];
                    arr[i + 1] = temp;
                    swapped = true;
                }
            }
            //不再发生位置交换说明已经完成排序，则结束循环
            if (!swapped) {
                break;
            }
            System.out.println("第" + (j + 1) + "轮冒泡:" + Arrays.toString(arr));
        }
    }
}
```

代码的运行结果如下所示：

```java
每轮比较次数：1 2 3 4 5 6 7 第1轮冒泡:[5, 3, 4, 2, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 5 6 第2轮冒泡:[3, 4, 2, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 5 第3轮冒泡:[3, 2, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 第4轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 2 3
```

> **说明**：由以上运行结果可知，优化后的代码效率明显更高，只要完成排序，就结束循环，不再进行无意义的比较。

##### 10.2.4 最终版

以上优化版的冒泡算法已经算是比较完美了，但还是有可优化的空间。比如我们要进行从小到大排序，但原数组中最大的几个数已经在末尾部分，并满足从小到大的要求了，这时候未排序好的数就没必要再和末尾的这几个数进行比较了。这样优化之后，每轮冒泡的比较次数就会有所减少了，优化后的最终版代码如下：

```java
package com.gongsl.test.array;

import java.util.Arrays;
import java.util.concurrent.atomic.AtomicInteger;

/**
 * @Author: gongsl
 * @Date: 2022-02-03 18:54
 */
public class BubbleSort {
    
    //这里的AtomicInteger只是为了当个计数器用
    private static final AtomicInteger atomic = new AtomicInteger(0);

    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        bubbleSort(array);
    }

    public static void bubbleSort(int[] arr) {
        int n = arr.length - 1;
        while (true) {
            System.out.print("每轮比较次数：");
            int last = 0;
            for (int i = 0; i < n; i++) {
                System.out.print((i + 1) + " ");
                if (arr[i] > arr[i + 1]) {
                    int temp = arr[i];
                    arr[i] = arr[i + 1];
                    arr[i + 1] = temp;
                    last = i;
                }
            }
            System.out.println("第"+atomic.incrementAndGet()+"轮冒泡:"+Arrays.toString(arr));
            n = last;
            if (n == 0) {
                break;
            }
        }
    }
}
```

运行结果如下所示：

```java
每轮比较次数：1 2 3 4 5 6 7 第1轮冒泡:[5, 3, 4, 2, 6, 7, 8, 9]
每轮比较次数：1 2 3 4 第2轮冒泡:[3, 4, 2, 5, 6, 7, 8, 9]
每轮比较次数：1 2 第3轮冒泡:[3, 2, 4, 5, 6, 7, 8, 9]
每轮比较次数：1 第4轮冒泡:[2, 3, 4, 5, 6, 7, 8, 9]
```

> **说明**：通过对**最终版**代码的运行结果和**优化版**代码的运行结果相比，可以发现，同样是进行了4轮冒泡后完成排序，但是**最终版**代码的运行结果每轮冒泡的比较次数明显减少。

#### 10.3 选择排序法

相关代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-03 21:51
 */
public class SelectionSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        selectionSort(array);
        System.out.println("最终结果：" + Arrays.toString(array));
    }

    public static void selectionSort(int[] arr) {
        for (int j = 0; j < arr.length - 1; j++) {
            //j代表每轮选择最小元素要交换到的目标索引
            int s = j;
            for (int i = s + 1; i < arr.length; i++) {
                if (arr[s] > arr[i]) {
                    s = i;
                }
            }
            if (s != j) {
                int temp = arr[s];
                arr[s] = arr[j];
                arr[j] = temp;
            }
        }
    }
}
```

> **说明**：选择排序法一般要快于冒泡排序法，因为其交换次数少；但如果集合有序度高，则冒泡排序优于选择排序。

#### 10.4 插入排序法

相关代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-04 11:43
 */
public class InsertionSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        insertionSort(array);
        System.out.println("最终结果：" + Arrays.toString(array));
    }

    public static void insertionSort(int[] arr) {
        for (int i = 1; i < arr.length; i++) {
            int t = arr[i]; // 代表待插入的元素值
            int j = i - 1; // 代表已排序区域的元素索引
            while (j >= 0) {
                if (t < arr[j]) {
                    arr[j + 1] = arr[j];
                } else {
                    break; // 退出循环，减少比较次数
                }
                j--;
            }
            arr[j + 1] = t;
        }
    }
}
```

> **说明**：大部分情况下，插入排序是优于选择排序和冒泡排序的；另外，对于有序集合，插入排序效率更高。

#### 10.5 快速排序法

##### 10.5.1 单边循环快排

###### 10.5.1.1 效果演示

单边循环快排的效果演示[动图](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220221155740.gif)如下所示：

![单边循环快排](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220221155740.gif) 

###### 10.5.1.2 具体代码

相关代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-04 19:08
 */
public class QuickSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 3, 4, 2, 6, 8, 9};
        quickSort(array, 0, array.length - 1);
        System.out.println("最终结果：" + Arrays.toString(array));
    }

    public static void quickSort(int[] arr, int l, int h) {
        if (l >= h) {
            return;
        }
        //返回值代表了基准点元素所在的正确索引，用它确定下一轮分区的边界
        int p = partition(arr, l, h);
        //左边分区的范围确定
        quickSort(arr, l, p - 1);
        //右边分区的范围确定
        quickSort(arr, p + 1, h);
    }

    public static int partition(int[] arr, int l, int h) {
        int pv = arr[h]; // 基准点元素
        int i = l;
        for (int j = l; j < h; j++) {
            if (arr[j] < pv) {
                if (i != j) {
                    int temp = arr[i];
                    arr[i] = arr[j];
                    arr[j] = temp;
                }
                i++;
            }
        }
        if (i != h) {
            int temp = arr[h];
            arr[h] = arr[i];
            arr[i] = temp;
        }
        return i;
    }
}
```

##### 10.5.2 双边循环快排

相关代码如下所示：

```java
package com.gongsl.test.array;

import java.util.Arrays;

/**
 * @Author: gongsl
 * @Date: 2022-02-04 19:08
 */
public class QuickSort {
    public static void main(String[] args) {
        int[] array = {5, 7, 9, 4, 2, 6, 8, 3};
        quickSort(array, 0, array.length - 1);
        System.out.println("最终结果：" + Arrays.toString(array));
    }

    public static void quickSort(int[] arr, int l, int h) {
        if (l >= h) {
            return;
        }
        //返回值代表了基准点元素所在的正确索引，用它确定下一轮分区的边界
        int p = partition(arr, l, h);
        //左边分区的范围确定
        quickSort(arr, l, p - 1);
        //右边分区的范围确定
        quickSort(arr, p + 1, h);
    }

    public static int partition(int[] arr, int l, int h) {
        int pv = arr[l]; // 基准点元素
        int i = l;
        int j = h;
        while (i < j) {
            //j从右往左找小的
            while (i < j && arr[j] > pv) {
                j--;
            }
            //i从左往右找大的
            while (i < j && arr[i] <= pv) {
                i++;
            }
            int temp = arr[i];
            arr[i] = arr[j];
            arr[j] = temp;
        }
        int temp = arr[l];
        arr[l] = arr[j];
        arr[j] = temp;
        return j;
    }
}
```

**注意事项：**

- 单边循环的快速排序法是选择最右边的元素作为基准点元素的；
- 双边循环的快速排序法是选择最左边的元素作为基准点元素的；
- 在数据量较大时，使用快速排序法的优势是非常明显的。

#### 10.6 时间复杂度比较

常见算法的时间复杂度对照表如下所示：

| 排序算法 | 时间复杂度`(最好)` | 时间复杂度`(平均)` | 时间复杂度`(最坏)` | 空间复杂度 | 稳定性 |
| :------: | :----------------: | :----------------: | :----------------: | :--------: | :----: |
| 冒泡排序 |        O(n)        |      O(n^2^)       |      O(n^2^)       |    O(1)    |  稳定  |
| 选择排序 |      O(n^2^)       |      O(n^2^)       |      O(n^2^)       |    O(1)    | 不稳定 |
| 插入排序 |        O(n)        |      O(n^2^)       |      O(n^2^)       |    O(1)    |  稳定  |
| 快速排序 |    O(nlog~2~n)     |    O(nlog~2~n)     |      O(n^2^)       | O(log~2~n) | 不稳定 |

> **说明**：以上算法的时间复杂度由小到大的顺序是：O(1) < O(log~2~n) < O(n) < O(nlog~2~n) < O(n^2^)，时间复杂度越小，算法的执行效率越低。

