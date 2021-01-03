#### 前言：导入的jar包

```xml
<dependency>
    <groupId>com.alibaba</groupId>
    <artifactId>fastjson</artifactId>
    <version>1.2.47</version>
</dependency>

<dependency>
    <groupId>org.projectlombok</groupId>
    <artifactId>lombok</artifactId>
    <version>1.16.18</version>
    <scope>compile</scope>
</dependency>
```

#### 1.字符串形式展示集合中的所有内容

工作中经常会遇到需要展示一个对象集合中所有内容的场景，比如打日志排查问题的时候。如果每次都对集合进行遍历，不仅代码繁多，还可能出错。这时可以把集合转换成字符串形式进行展示。

##### 1.1 实体类

```java
package com.gongsl.pojo;

import com.alibaba.fastjson.annotation.JSONType;
import lombok.AllArgsConstructor;
import lombok.Data;

/**
 * @Author: gongsl
 * @Date: 2020-04-27 16:28
 */
@Data
@AllArgsConstructor
@JSONType(orders = {"id","name","age","gender","isAdult"})
public class Person{

    //编号
    private Integer id;

    //姓名
    private String name;

    //年龄
    private Integer age;

    //性别
    private String gender;

    //是否成年
    private Boolean isAdult;
}
```

##### 1.2 测试代码

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSON;
import com.gongsl.pojo.Person;
import java.util.ArrayList;
import java.util.List;

/**
 * @Author: gongsl
 * @Date: 2020-12-28 22:04
 */
public class SkillTest {
    public static void main(String[] args) {
        List<Person> list = new ArrayList<Person>(){
            {
                add(new Person(1,"张三",12,"男",false));
                add(new Person(2,"李四",25,"女",true));
            }
        };
        System.out.println(JSON.toJSONString(list,true));
    }
}
```

##### 1.3 代码运行结果

```json
[
	{
		"id":1,
		"name":"张三",
		"age":12,
		"gender":"男",
		"isAdult":false
	},
	{
		"id":2,
		"name":"李四",
		"age":25,
		"gender":"女",
		"isAdult":true
	}
]
```

`测试代码中调用的是JSON类的toJSONString(Object object, boolean prettyFormat)方法，该方法的第二个参数的值如果为true的话，表示会对方法返回的字符串进行格式化处理。如果不加这个参数，即调用toJSONString(Object object)方法的话，运行结果就是下面这个。`

```json
[{"id":1,"name":"张三","age":12,"gender":"男","isAdult":false},{"id":2,"name":"李四","age":25,"gender":"女","isAdult":true}]
```

##### 1.4 关于字符串中节点顺序的问题

如果使用的是`com.alibaba.fastjson.JSON`类中的`toJSONString`方法进行对象到字符串的转换的话，转换后的字符串中的节点在显示的时候默认是无序的。如果我们想要它们按照指定顺序显示，可以使用`@JSONType`注解来实现。比如上面的Person类，类上面加上`@JSONType(orders = {"id","name","age","gender","isAdult"})`这一行内容后，转换后的字符串中的id、name等节点就会按照注解中指定的顺序进行显示，最先指定的会显示在最前面，比如id会显示在第一位，name会显示在第二位，然后依次类推。