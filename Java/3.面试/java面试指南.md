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

##### 2.4.1 两者的异同

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

##### 2.4.2 关于公平锁

* 公平锁的公平体现：
  * **已经处在阻塞队列**中的线程(不考虑超时)始终都是公平的，先进先出；
  * 公平锁是指**未处于阻塞队列**中的线程来争抢锁，如果队列不为空，则老实到队尾等待；
  * 非公平锁是指**未处于阻塞队列**中的线程来争抢锁，与队列头唤醒的线程去竞争，谁抢到算谁的。
* 公平锁会降低吞吐量，一般不用。

##### 2.4.3 关于条件变量

* ReentrantLock中的条件变量功能类似于普通synchronized的wait，notify，用在当线程获得锁后，发现条件不满足时，临时等待的链表结构；
* 与synchronized的等待集合不同之处在于，ReentrantLock中的条件变量可以有多个，可以实现更精细的等待、唤醒控制。

#### 2.5 volatile关键字

##### 2.5.1 线程安全的三个特性

线程安全要考虑三个方面：`可见性`、`有序性`、`原子性`。

- 可见性：指一个线程对共享变量修改，另一个线程能看到最新的结果；
- 有序性：指一个线程内代码按编写顺序执行；
- 原子性：指一个线程内多行代码以一个整体运行，期间不能有其它线程的代码插队。

##### 2.5.2 三个特性的起因和解决办法

**可见性：**

* 起因：由于**编译器优化、或缓存优化、或CPU指令重排序优化**导致的对共享变量所做的修改另外的线程看不到。
* 解决：用volatile修饰共享变量，能够防止编译器等优化发生，让一个线程对共享变量的修改对另一个线程可见。

**有序性：**

* 起因：由于**编译器优化、或缓存优化、或CPU指令重排序优化**导致指令的实际执行顺序与编写顺序不一致。
* 解决：用volatile修饰共享变量会在读、写共享变量时加入不同的屏障，阻止其他读写操作越过屏障，从而达到阻止重排序的效果。

**原子性：**

* 起因：多线程下，不同线程的**指令发生了交错**导致的共享变量的读写混乱。
* 解决：用悲观锁或乐观锁解决，volatile并不能解决原子性。

##### 2.5.3 volatile能否保证线程安全

> volatile能够保证共享变量的**可见性**和**有序性**，但是并不能保证**原子性**。

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

#### 2.7 Hashtable和ConcurrentHashMap

##### 2.7.1 两者的异同

* Hashtable与ConcurrentHashMap都是线程安全的Map集合；
* Hashtable并发度低，整个Hashtable对应一把锁，同一时刻，只能有一个线程操作它；
* ConcurrentHashMap并发度高，整个ConcurrentHashMap对应多把锁，只要线程访问的是不同锁，就不会冲突；
* 从jdk1.8开始，ConcurrentHashMap将数组的每个头节点作为锁，如果多个线程访问的头节点不同，则不会冲突。

> **说明**：从jdk1.8开始，ConcurrentHashMap底层结构也是采用的数组+链表+红黑树，所以数组中的每个元素其实都是头节点，然后每个元素下面根据实际情况可能会存在链表或者红黑树。

##### 2.7.2 jdk1.7中的ConcurrentHashMap

* 数据结构：`Segment(大数组) + HashEntry(小数组) + 链表`，每个Segment对应一把锁，如果多个线程访问不同的 Segment，则不会冲突；
* 并发度：Segment数组大小即并发度，决定了同一时刻最多能有多少个线程并发访问。Segment数组不能扩容，意味着并发度在ConcurrentHashMap创建时就固定了；
* 扩容：每个小数组的扩容相对独立，小数组在超过扩容因子时会触发扩容，每次扩容翻倍；
* Segment[0]原型：首次创建其它小数组时，会以此原型为依据，数组长度，扩容因子都会以原型为准。

##### 2.7.3 jdk1.8中的ConcurrentHashMap

* 数据结构：`Node数组 + 链表 + 红黑树`，数组的每个头节点作为锁，如果多个线程访问的头节点不同，则不会冲突。首次生成头节点时如果发生竞争，会利用cas而非syncronized，进一步提升性能；
* 并发度：Node数组有多大，并发度就有多大，与jdk1.7的Segment(大数组)不同，jdk1.8的Node数组可以扩容；
* 扩容条件：Node数组满3/4时就会扩容；
* 扩容过程：以链表为单位从后向前迁移链表，迁移完成的将旧数组头节点替换为ForwardingNode，代码Node数组中的这个节点已经迁移完成；
* 扩容时并发get：
  * 根据是否为ForwardingNode来决定是在新数组查找还是在旧数组查找，不会阻塞；
  * 如果链表长度超过1，则需要对节点进行复制，即创建新节点；
  * 如果链表最后几个元素扩容后索引不变，则节点无需复制。
* 扩容时并发put：
  * 如果put的线程与扩容线程操作的链表是同一个，put线程会阻塞；
  * 如果put的线程操作的链表还未迁移完成，即头节点不是ForwardingNode，则可以并发执行；
  * 如果put的线程操作的链表已经迁移完成，即头节点是ForwardingNode，则可以协助扩容。
* 源码中的capacity属性值代表的是预估的元素个数，不是数组的初始大小；
* loadFactor只在计算初始数组大小时被使用，之后扩容因子固定为3/4；
* 超过树化阈值时的扩容问题，如果容量已经是64，直接树化，否则在原来容量基础上做3轮扩容。

#### 2.8 ThreadLocal的用法

##### 2.8.1 ThreadLocal的作用

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

##### 2.8.2 ThreadLocal的原理

在ThreadLocal的底层其实是通过内部类`ThreadLocalMap`来存储资源对象的：

- 调用ThreadLocal类的set方法时，其实就是以ThreadLocal自己作为key，资源对象作为value，放入当前线程的ThreadLocalMap集合中；
- 调用get方法时，就是以ThreadLocal自己作为key，到当前线程中查找关联的资源值；
- 调用remove方法时，就是以ThreadLocal自己作为key，移除当前线程关联的资源值。

**关于ThreadLocalMap的一些特点：**

- key的hash值统一分配；
- 初始容量是16，扩容因子为2/3，扩容后容量翻倍；
- key索引冲突后用开放寻址法解决冲突；
- ThreadLocalMap中的key被设计为弱引用，当key不再使用的时候，如果内存不足，便于释放其占用的内存。

##### 2.8.3 内存释放时机

- ThreadLocalMap中的key被设计为弱引用，所以不再使用的时候是可以被释放的，不过仅仅会让key的内存释放，关联的value的内存并不会得到释放；
- 我们可以通过调用get方法释放value内存，因为如果key已经被释放，使用get方法时就会发现key值为null，这时候就会释放其value的内存；
- 或者我们也可以使用set方法通过启发式扫描释放value内存，如果key值已经被释放，再使用set方法，不仅会释放这个key对应value的内存，也会清除临近key值为null的value的内存；
- 我们也可以主动使用remove方法释放key、value的内存：
  - 使用remove方法不仅可以使用key、value的内存，也会清除临近key值为null的value的内存；
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

### 4.Spring篇

#### 4.1 Spring事务失效的场景

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











### 5.Mysql篇





### 6.Redis篇





### 7.Linux篇







### 8.微服务篇





### 9.网络协议篇





### 10.算法篇





### 11.人事篇

































