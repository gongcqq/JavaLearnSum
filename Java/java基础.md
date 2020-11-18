### 1.变量之间的运算

#### 1.1 自动类型提升

当容量小的数据类型的变量与容量大的数据类型的变量做运算时，结果自动提升为容量大的数据类型。

`byte、char、short ---> int ---> long ---> float ---> double`

###### 当byte类型的数据和int类型的数据做运算时：

**案例一：**

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        int i = 3;
        int result = b + i;//这里的类型也可以是long，但至少要是int，不能是byte或者short
        System.out.println("result=" + result);//result=5
    }
}
```

**案例二：**

```java
public class Test {
    public static void main(String[] args) {
        byte b = 2;
        int result = b + 3;//这里的数字3默认是int类型，和int类型做运算，返回值类型至少要是int
        System.out.println("result=" + result);//result=5
    }
}
```



特殊情况：**当byte、char、short三种类型的变量互相之间做运算的时候，结果均为int类型。**

