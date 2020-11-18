### 1.变量之间的运算

#### 1.1 自动类型提升

当容量小的数据类型的变量与容量大的数据类型的变量做运算时，结果自动提升为容量大的数据类型。

`byte、char、short ---> int ---> long ---> float ---> double`

**当byte类型的数据和int类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        int i = 3;
        //这里的类型也可以是long，但至少要是int，不能是byte或者short
        int result = b + i;
        System.out.println("result=" + result);//result=5
    }
}
```

*案例二：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        //这里的数字3默认是int类型，和int类型做运算，返回值类型至少要是int
        int result = b + 3;
        System.out.println("result=" + result);//result=5
    }
}
```

**当byte类型的数据和float类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        float f = 1.2f;
        //这里的类型也可以是double，但至少要是float
        float result = b + f;
        //如果result的类型是double的话，那么这里result的值就是"3.200000047683716"
        System.out.println("result=" + result);//result=3.2
    }
}
```

*案例二：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        //这里的数字1.2默认是double类型，所以返回值类型只能是double类型，不能是float类型
        double result = b + 1.2;
        System.out.println("result=" + result);//result=3.2
    }
}
```

**当byte类型的数据和double类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        double d = 1.2;
        //这里result的类型只能是double类型，不能是float类型
        double result = b + d;
        System.out.println("result=" + result);//result=3.2
    }
}
```

**当char类型的数据和int类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        char c = 'a';
        int i = 3;
        //字符'a'对应的ASCII码是97，所以result的值为100，返回值的类型也能是long类型，但至少要是int类型
        int result = c + i;
        System.out.println("result=" + result);//result=100
    }
}
```

**当byte类型的数据和short类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        short s = 3;
        //这里result的类型也可以是long类型，但至少要是int类型，不能是byte类型或者short类型
        int result = b + s;
        System.out.println("result=" + result);//result=5
    }
}
```

**当byte类型的数据和char类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte b = 3;
        char c = 'a';
        //这里result的类型可以是long类型，但至少要是int类型，不能是byte类型或者char类型
        int result = b + c;
        System.out.println("result=" + result);//result=100
    }
}
```

**当char类型的数据和short类型的数据做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        char c = 'a';
        short s = 3;
        //这里result的类型可以是long类型，但至少要是int类型，不能是char类型或者short类型
        int result = c + s;
        System.out.println("result=" + result);//result=100
    }
}
```

**当byte类型、char类型和short类型的数据分别与自己做运算时：**

*案例一：*

```java
public class Test {
    public static void main(String[] args) {
        byte a = 2;
        byte b = 3;
        //这里result的类型可以是long类型，但至少要是int类型，不能是byte类型
        int result = a + b;
        System.out.println("result=" + result);//result=5
    }
}
```

*案例二：*

```java
public class Test {
    public static void main(String[] args) {
        char a = 'a';
        char b = 'b';
        //这里result的类型可以是long类型，但至少要是int类型，不能是char类型
        int result = a + b;
        System.out.println("result=" + result);//result=195
    }
}
```

*案例三：*

```java
public class Test {
    public static void main(String[] args) {
        short a = 2;
        short b = 3;
        //这里result的类型可以是long类型，但至少要是int类型，不能是short类型
        int result = a + b;
        System.out.println("result=" + result);//result=5
    }
}
```

**<font color="red">注意：</font>当byte、char、short三种类型的变量互相之间做运算的时候，结果均为int类型。**

#### 1.2 强制类型转换







