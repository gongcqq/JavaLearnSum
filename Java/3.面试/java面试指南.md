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

1. `REQUIRED`：如果当前没有事务，则新建一个事务，如果当前存在事务，则加入这个事务，这个是默认的传播机制。
2. `SUPPORTS`：当前存在事务，则加入当前事务，如果当前没有事务，则以非事务的方式执行。
3. `MANDATORY`：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常。
4. `REQUIRES_NEW`：总是创建一个新事务，如果当前已经存在事务，则挂起已经存在的事务。
5. `NOT_SUPPORTED`：以非事务方式执行，如果存在当前事务，则挂起当前事务。
6. `NEVER`：不使用事务，如果当前存在事务，则抛出异常。
7. `NESTED`：如果当前存在事务，则在嵌套事务内执行，如果当前没有事务，则执行与**REQUIRED**一样的操作。

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
- `REPEATABLE_READ`(可重复读)：该级别能够避免脏读、不可重复读，但是不能避免幻读；
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

   ![image-20220105165300556](https://cdn.jsdelivr.net/gh/gongcqq/FigureBed@main/Image/Typora/20220106231317.png) 

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

![image-20220107172710622](D:\Program Files (x86)\Typora\images\java面试指南\image-20220107172710622.png) 

我们也可以直接使用`show variables like '%default_storage_engine%';`命令查看默认的存储引擎：

![image-20220107173017552](D:\Program Files (x86)\Typora\images\java面试指南\image-20220107173017552.png) 

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

![image-20220107225113087](D:\Program Files (x86)\Typora\images\java面试指南\image-20220107225113087.png) 

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

我们也可以通过以下语句显示地给记录集加共享锁或排它锁：

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























### 6.Redis篇





### 7.Linux篇







### 8.微服务篇





### 9.网络协议篇





### 10.算法篇





### 11.人事篇

































