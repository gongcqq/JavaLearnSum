### 1.使用前的准备

#### 1.1 jdk版本

代码中使用的是jdk1.8版本。

#### 1.2 导入jar包

```xml
<dependency>
   <groupId>com.google.guava</groupId>
   <artifactId>guava</artifactId>
   <version>23.0</version>
</dependency>
```

### 2.Joiner类的常用方法

#### 2.1 字符串的拼接

```java
package com.gongsl.test;

import com.google.common.base.Joiner;
import java.util.Arrays;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jack", "Bob", "Jerry", "Lucy");
        String str = Joiner.on("|").join(list);
        System.out.println(str);//运行结果：Tom|Jack|Bob|Jerry|Lucy
    }
}
```

#### 2.2 字符串的拼接(跳过null值)

```java
package com.gongsl.test;

import com.google.common.base.Joiner;
import java.util.Arrays;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jack", "Bob", null, "Lucy");
        String str = Joiner.on("|").skipNulls().join(list);
        System.out.println(str);//运行结果：Tom|Jack|Bob|Lucy
    }
}
```

#### 2.3 字符串的拼接(给null值设置默认值)

```java
package com.gongsl.test;

import com.google.common.base.Joiner;
import java.util.Arrays;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        List<String> list = Arrays.asList("Tom", "Jack", "Bob", null, "Lucy");
        String str = Joiner.on("|").useForNull("default").join(list);
        System.out.println(str);//运行结果：Tom|Jack|Bob|default|Lucy
    }
}
```

`注：以上三种场景都是拿集合来举例的，如果换做数组，也都可以达到同样的效果。`

### 3.Splitter类的常用方法

#### 3.1 字符串的分割

```java
package com.gongsl.test;

import com.google.common.base.Splitter;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        String str = "Tom|Jack|Bob|Lucy|Jerry";
        List<String> list = Splitter.on("|").splitToList(str);
        System.out.println(list);//运行结果：[Tom, Jack, Bob, Lucy, Jerry]
    }
}
```

#### 3.2 字符串的分割(跳过空值)

```java
package com.gongsl.test;

import com.google.common.base.Splitter;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        String str = "Tom|Jack|Bob||Lucy|Jerry";
        List<String> list = Splitter.on("|").omitEmptyStrings().splitToList(str);
        System.out.println(list);//运行结果：[Tom, Jack, Bob, Lucy, Jerry]
    }
}
//如果上面不使用omitEmptyStrings()方法，运行结果就是：[Tom, Jack, Bob, , Lucy, Jerry]
```

#### 3.3 字符串的分割(去除空格)

```java
package com.gongsl.test;

import com.google.common.base.Splitter;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        String str = "Tom|Jack|Bob  |Lucy|Jerry";
        List<String> list = Splitter.on("|").trimResults().splitToList(str);
        System.out.println(list);//运行结果：[Tom, Jack, Bob, Lucy, Jerry]
    }
}
//如果上面不使用trimResults()方法，运行结果就是：[Tom, Jack, Bob  , Lucy, Jerry]
```

#### 3.4 根据指定长度分割字符串

```java
package com.gongsl.test;

import com.google.common.base.Splitter;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        String str = "abcdefghijklmn";
        List<String> list = Splitter.fixedLength(3).splitToList(str);
        System.out.println(list);//运行结果：[abc, def, ghi, jkl, mn]
    }
}
```

#### 3.5 指定字符串分割后的最大个数

```java
package com.gongsl.test;

import com.google.common.base.Splitter;
import java.util.List;

public class GuavaTest {
    public static void main(String[] args) {
        String str = "Tom|Jack|Bob|Lucy|Jerry";
        List<String> list = Splitter.on("|").limit(3).splitToList(str);
        System.out.println(list);//运行结果：[Tom, Jack, Bob|Lucy|Jerry]
    }
}
//这里的意思就是根据“|”把字符串进行分割，分割后的集合的个数最大只能是3个。
```



























