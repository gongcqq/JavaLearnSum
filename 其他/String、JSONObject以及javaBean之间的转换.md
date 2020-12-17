#### 1.导入的jar包

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

#### 2.实体类

```java
package com.gongsl.notebook.copy.pojo;

import lombok.Data;

/**
 * @Author: gongsl
 * @Date: 2020-04-27 16:28
 */
@Data
public class Person{

    //编号
    private int id;

    //姓名
    private String name;

    //年龄
    private String age;

    //性别
    private String gender;
}
```

#### 3.转换的逻辑

##### 3.1 String--->javaBean

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;
import com.gongsl.notebook.copy.pojo.Person;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        String str = "{\"id\":1,\"name\":\"张三\",\"age\":\"25\",\"gender\":\"男\"}";
        
        //String--->javaBean
        Person person = JSONObject.parseObject(str, Person.class);
        System.out.println(person);
    }
}

//运行结果---> Person(id=1, name=张三, age=25, gender=男)
```

##### 3.2 String--->JSONObject

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        String str = "{\"id\":1,\"name\":\"张三\",\"age\":\"25\",\"gender\":\"男\"}";
        
        //String--->JSONObject
        JSONObject jsonObject = JSONObject.parseObject(str);
        System.out.println(jsonObject);
    }
}

//运行结果---> {"id":1,"age":"25","name":"张三","gender":"男"}
```

##### 3.3 javaBean--->String

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;
import com.gongsl.notebook.copy.pojo.Person;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        Person person = new Person();
        person.setId(1);
        person.setName("张三");
        person.setAge("25");
        person.setGender("男");
        
        //javaBean--->String
        String str = JSONObject.toJSONString(person);
        System.out.println(str);
    }
}

//运行结果---> {"age":"25","gender":"男","id":1,"name":"张三"}
```

##### 3.4 JSONObject--->String

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("id",1);
        jsonObject.put("name","张三");
        jsonObject.put("age","25");
        jsonObject.put("gender","男");
        
        //JSONObject--->String
        String str = jsonObject.toString();
        System.out.println(str);
    }
}

//运行结果---> {"id":1,"age":"25","name":"张三","gender":"男"}
```

##### 3.5 javaBean--->JSONObject

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;
import com.gongsl.notebook.copy.pojo.Person;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        Person person = new Person();
        person.setId(1);
        person.setName("张三");
        person.setAge("25");
        person.setGender("男");
        
        //javaBean--->JSONObject
        JSONObject jsonObject = (JSONObject) JSONObject.toJSON(person);
        System.out.println(jsonObject);
    }
}

//运行结果---> {"id":1,"name":"张三","age":"25","gender":"男"}
```

##### 3.6 JSONObject--->javaBean

```java
package com.gongsl.test;

import com.alibaba.fastjson.JSONObject;
import com.gongsl.notebook.copy.pojo.Person;

/**
 * @Author: gongsl
 * @Date: 2020-11-25 23:48
 */
public class JsonTest {
    public static void main(String[] args) {
        JSONObject jsonObject = new JSONObject();
        jsonObject.put("id",1);
        jsonObject.put("name","张三");
        jsonObject.put("age","25");
        jsonObject.put("gender","男");

        //JSONObject--->javaBean
        Person person = JSONObject.toJavaObject(jsonObject, Person.class);
        System.out.println(person);
    }
}

//运行结果---> Person(id=1, name=张三, age=25, gender=男)
```

