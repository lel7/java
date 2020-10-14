使用：

### List

##### haskey

```java
if(redisTemplate.haKey("test")){
    sout("Y");
}else {
    sout("N");
}
```



##### range

用于从redis缓存中获取指定区间的数据

```java
if(redisTemplate.hasKey("test")){
    //该键的值为 [4, 3, 2, 1]
     System.out.println(redisTemplate.opsForList().range("test", 0, 0)); // [4]
    System.out.println(redisTemplate.opsForList().range("test", 0, 1)); // [4, 3]
    System.out.println(redisTemplate.opsForList().range("test", 0, 2)); // [4, 3, 2]
    System.out.println(redisTemplate.opsForList().range("test", 0, 3)); // [4, 3, 2, 1]
    System.out.println(redisTemplate.opsForList().range("test", 0, 4)); // [4, 3, 2, 1]
    System.out.println(redisTemplate.opsForList().range("test", 0, 5)); // [4, 3, 2, 1]
    
    System.out.println(redisTemplate.opsForList().range("test", 0, -1)); // [4, 3, 2, 1] 如果结束位是-1， 则表示取所有的值
}
```



##### delete

```java
List<String> test = new ArrayList<>();
test.add("1");
test.add("2");
test.add("3");
test.add("4");

redisTemplate.opsForList().rightPushAll("test", test);
System.out.println(redisTemplate.opsForList().range("test", 0, -1)); // [1, 2, 3, 4]
redisTemplate.delete("test");
System.out.println(redisTemplate.opsForList().range("test", 0, -1)); // []
```



##### size

```java
System.out.println(redisTemplate.opsForList().size("test")); // 4
```



leftpush  ||  rightpush

```java
for (int i = 0; i < 4; i++) {
    Integer value = i + 1;
    redisTemplate.opsForList().leftPush("test", value.toString());
    System.out.println(redisTemplate.opsForList().range("test", 0, -1));
}
```

```
[1]
[2, 1]
[3, 2, 1]
[4, 3, 2, 1]

```

##### leftPushAll

基本和leftPush一样，只不过是一次性的将List入栈。



##### leftPushIfPresent

跟`leftPush`是同样的操作，唯一的不同是，当且仅当key存在时，才会更新key的值。如果key不存在则不会对数据进行任何操作。

```java
redisTemplate.delete("test");

redisTemplate.opsForList().leftPushIfPresent("test", "1");
redisTemplate.opsForList().leftPushIfPresent("test", "2");
System.out.println(redisTemplate.opsForList().range("test", 0, -1)); // []

```

##### leftPop

该函数用于移除上面我们抽象的容器中的最左边的一个元素。

```java
List<String> test = new ArrayList<>();
test.add("1");
test.add("2");
test.add("3");
test.add("4");
redisTemplate.opsForList().rightPushAll("test", test);

redisTemplate.opsForList().leftPop("test"); // [2, 3, 4]
redisTemplate.opsForList().leftPop("test"); // [3, 4]
redisTemplate.opsForList().leftPop("test"); // [4]
redisTemplate.opsForList().leftPop("test"); // []
redisTemplate.opsForList().leftPop("test"); // []

```

值得注意的是，当返回为空后，在redis中这个key也不复存在了。如果此时再调用[leftPushIfPresent](https://juejin.im/post/6844903763354845191#heading-10)，是无法再添加数据的。有代码有真相。





##### index

获取list中指定位置的元素。



```java
if (redisTemplate.hasKey("test")) {
    // 该键的值为 [1, 2, 3, 4]
    System.out.println(redisTemplate.opsForList().index("test", -1)); // 4
    System.out.println(redisTemplate.opsForList().index("test", 0)); // 1
    System.out.println(redisTemplate.opsForList().index("test", 1)); // 2
    System.out.println(redisTemplate.opsForList().index("test", 2)); // 3
    System.out.println(redisTemplate.opsForList().index("test", 3)); // 4
    System.out.println(redisTemplate.opsForList().index("test", 4)); // null
    System.out.println(redisTemplate.opsForList().index("test", 5)); // null
}

```

值得注意的有两点。一个是如果下标是`-1`的话，则会返回List最后一个元素，另一个如果数组下标越界，则会返回`null`



##### trim

用于截取指定区间的元素，

。`range`是获取指定区间内的数据，而`trim`是留下指定区间的数据，删除不在区间的所有数据。`trim`是`void`，不会返回任何数据。

```JAVA
List<String> test = new ArrayList<>();
test.add("1");
test.add("2");
test.add("3");
test.add("4");
redisTemplate.opsForList().rightPushAll("test", test); // [1, 2, 3, 4]

redisTemplate.opsForList().trim("test", 0, 2); // [1, 2, 3]

```



##### remove

用于移除键中指定的元素。接受3个参数，分别是缓存的键名，计数事件，要移除的值。计数事件可以传入的有三个值，分别是`-1`、`0`、`1`。

`-1`代表从存储容器的最右边开始，删除一个与要移除的值匹配的数据；`0`代表删除所有与传入值匹配的数据；`1`代表从存储容器的最左边开始，删除一个与要移除的值匹配的数据。

```java
List<String> test = new ArrayList<>();
test.add("1");
test.add("2");
test.add("3");
test.add("4");
test.add("4");
test.add("3");
test.add("2");
test.add("1");

redisTemplate.opsForList().rightPushAll("test", test); // [1, 2, 3, 4, 4, 3, 2, 1]

// 当计数事件是-1、传入值是1时
redisTemplate.opsForList().remove("test", -1, "1"); // [1, 2, 3, 4, 4, 3, 2]

// 当计数事件是1，传入值是1时
redisTemplate.opsForList().remove("test", 1, "1"); // [2, 3, 4, 4, 3, 2]

// 当计数事件是0，传入值是4时
redisTemplate.opsForList().remove("test", 0, "4"); // [2, 3, 3, 2]

```

### Hash

存储类型为hash其实很好理解。在上述的`List`中，一个redis的Key可以理解为一个List，而在`Hash`中，一个redis的Key可以理解为一个HashMap。

##### put

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString()); // [1, 2, 3, 4]
redisTemplate.opsForHash().put("test", "isAdmin", true); // true

```



##### putAll

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
List<String> list2 = new ArrayList<>();
list2.add("5");
list2.add("6");
list2.add("7");
list2.add("8");
Map<String, String> valueMap = new HashMap<>();
valueMap.put("map1", list.toString());
valueMap.put("map2", list2.toString());

redisTemplate.opsForHash().putAll("test", valueMap); // {map2=[5, 6, 7, 8], map1=[1, 2, 3, 4]}

```

##### putIfAbsent

用于向一个Hash键中写入数据。当key在Hash键中已经存在时，则不会写入任何数据，只有在Hash键中不存在这个key时，才会写入数据。

同时，如果连这个Hash键都不存在，redisTemplate会新建一个Hash键，再写入key。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
redisTemplate.opsForHash().putIfAbsent("test", "map", list.toString());
System.out.println(redisTemplate.opsForHash().entries("test")); // {map=[1, 2, 3, 4]}

```



##### get



值得注意的是，使用`get`函数获取的数据都是Object类型。

所以需要使用类型与上述例子中的布尔类型的话，则需要强制转换一次。`List`类型则可以使用`fastjson`这种工具来进行转换。转换的例子已列举在上述代码中。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

System.out.println(redisTemplate.opsForHash().get("test", "map")); // [1, 2, 3, 4]
System.out.println(redisTemplate.opsForHash().get("test", "isAdmin")); // true

Boolean bool = (Boolean) redisTemplate.opsForHash().get("test", "isAdmin");
System.out.println(bool); // true

String str = redisTemplate.opsForHash().get("test", "map").toString();
List<String> array = JSONArray.parseArray(str, String.class);
System.out.println(array.size()); // 4

```



##### delete

```java
 List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
List<String> list2 = new ArrayList<>();
list2.add("5");
list2.add("6");
list2.add("7");
list2.add("8");
Map<String, String> valueMap = new HashMap<>();
valueMap.put("map1", list.toString());
valueMap.put("map2", list2.toString());

redisTemplate.opsForHash().putAll("test", valueMap); // {map2=[5, 6, 7, 8], map1=[1, 2, 3, 4]}
redisTemplate.opsForHash().delete("test", "map1"); // {map2=[5, 6, 7, 8]}

```



##### values

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

System.out.println(redisTemplate.opsForHash().values("test")); // [[1, 2, 3, 4], true]

```



##### entries

用于以Map的格式获取一个Hash键的所有值。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

Map<String, String> map = redisTemplate.opsForHash().entries("test");
System.out.println(map.get("map")); // [1, 2, 3, 4]
System.out.println(map.get("map") instanceof String); // true
System.out.println(redisTemplate.opsForHash().entries("test")); // {a=[1, 2, 3, 4], isAdmin=true}

```



##### hasKey

用于获取一个Hash键中是否含有某个键。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

System.out.println(redisTemplate.opsForHash().hasKey("test", "map")); // true
System.out.println(redisTemplate.opsForHash().hasKey("test", "b")); // false
System.out.println(redisTemplate.opsForHash().hasKey("test", "isAdmin")); // true

```

##### keys

用于获取一个Hash键中所有的键。

```
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

System.out.println(redisTemplate.opsForHash().keys("test")); // [a, isAdmin]

```



##### size

用于获取一个Hash键中包含的键的数量。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");

redisTemplate.opsForHash().put("test", "map", list.toString());
redisTemplate.opsForHash().put("test", "isAdmin", true);

System.out.println(redisTemplate.opsForHash().size("test")); // 2

```



##### increment

用于让一个Hash键中的某个key，根据传入的值进行累加。传入的数值只能是`double`或者`long`，不接受浮点型

```java
redisTemplate.opsForHash().increment("test", "a", 3);
redisTemplate.opsForHash().increment("test", "a", -3);
redisTemplate.opsForHash().increment("test", "a", 1);
redisTemplate.opsForHash().increment("test", "a", 0);

System.out.println(redisTemplate.opsForHash().entries("test")); // {a=1}

```



##### multiGet

用于批量的获取一个Hash键中多个key的值。

```java
List<String> list = new ArrayList<>();
list.add("1");
list.add("2");
list.add("3");
list.add("4");
List<String> list2 = new ArrayList<>();
list2.add("5");
list2.add("6");
list2.add("7");
list2.add("8");

redisTemplate.opsForHash().put("test", "map1", list.toString()); // [1, 2, 3, 4]
redisTemplate.opsForHash().put("test", "map2", list2.toString()); // [5, 6, 7, 8]

List<String> keys = new ArrayList<>();
keys.add("map1");
keys.add("map2");

System.out.println(redisTemplate.opsForHash().multiGet("test", keys)); // [[1, 2, 3, 4], [5, 6, 7, 8]]
System.out.println(redisTemplate.opsForHash().multiGet("test", keys) instanceof List); // true

```



## 新建一个`RedisConfig`配置文件

```java
package com.detectivehlh;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

/**
 * RedisConfig
 *
 * @author Lunhao Hu
 * @date 2019-01-17 15:12
 **/
@Configuration
public class RedisConfig {
    @Bean
    public RedisTemplate<String, String> redisTemplate(RedisConnectionFactory factory) {
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        //redis序列化
        Jackson2JsonRedisSerializer jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer(Object.class);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        StringRedisTemplate template = new StringRedisTemplate(factory);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }
}

```

