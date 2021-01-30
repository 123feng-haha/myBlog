## 前言

目前解析Json的工具包有，Gson，FastJson，Jackson，Json-lib。综合来看，Jackson的性能较优，稳定性也比较高，而且spring-boot-starter-web默认会引入Jackson包。因此介绍一下Jackson的使用。

Jackson目前有2个版本
1.x版本包名为org.codehaus.jackson
2.x版本包名为com.fasterxml.jackson



## 使用

在pom中加入如下依赖即可。

```java
<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-core</artifactId>
	<version>2.9.2</version>
</dependency>

<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-databind</artifactId>
	<version>2.9.2</version>
</dependency>

<dependency>
	<groupId>com.fasterxml.jackson.core</groupId>
	<artifactId>jackson-annotations</artifactId>
	<version>2.9.2</version>
</dependency>
```

==注意：==当项目使用spring-boot-starter-web模块时，会默认引入Jackson包，不必在pom中再次引入上面依赖

```
<dependency>
	<groupId>org.springframework.boot</groupId>
	<artifactId>spring-boot-starter-web</artifactId>
</dependency>
```



## 序列化

将java对象序列化成json

1.将对象转为json

```java
@Slf4j
public class JsonUtil {

    private static ObjectMapper objectMapper = new ObjectMapper();
    
    /** 将对象转为string */
    public static <T> String obj2String(T obj){

        if(obj == null){
            return null;
        }
        try {
            return obj instanceof String ? (String)obj :  objectMapper.writeValueAsString(obj);
        } catch (Exception e) {
            log.warn("Parse Object to String error",e);
            return null;
        }
    }
}
```

2.将对象转为json，并格式化的输出

```java
public static <T> String obj2StringPretty(T obj){
	if(obj == null){
		return null;
	}
	try {
		return obj instanceof String ? (String)obj :  objectMapper.writerWithDefaultPrettyPrinter().writeValueAsString(obj);
	} catch (Exception e) {
		log.warn("Parse Object to String error",e);
		return null;
	}
}
```



## 反序列化

将json转为java对象

1.常用--满足大部分场景

```java
public static <T> T string2Obj(String str, Class<T> clazz){
	if(StringUtils.isEmpty(str) || clazz == null){
		return null;
	}

	try {
		return clazz.equals(String.class)? (T)str : objectMapper.readValue(str, clazz);
	} catch (Exception e) {
		log.warn("Parse String to Object error",e);
		return null;
	}
}
```



2.不常用

```java
public static <T> T string2Obj(String str, TypeReference<T> typeReference){
	if(StringUtils.isEmpty(str) || typeReference == null){
		return null;
	}
	try {
		return (T)(typeReference.getType().equals(String.class)? str : objectMapper.readValue(str,typeReference));
	} catch (Exception e) {
		log.warn("Parse String to Object error",e);
		return null;
	}
}
```



3.不常用

```java
public static <T> T string2Obj(String str, Class<?> collectionClass, Class<?>... elementClasses){
	JavaType javaType = objectMapper.getTypeFactory().constructParametricType(collectionClass,elementClasses);
	try {
		return objectMapper.readValue(str,javaType);
	} catch (Exception e) {
		log.warn("Parse String to Object error",e);
		return null;
	}
}
```





## jackson的配置

```java
private final static ObjectMapper objectMapper = new ObjectMapper();

    static {
        objectMapper.configure(JsonParser.Feature.ALLOW_COMMENTS, true);
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_FIELD_NAMES, true);
        objectMapper.configure(JsonParser.Feature.ALLOW_SINGLE_QUOTES, true);
        objectMapper.configure(JsonParser.Feature.ALLOW_UNQUOTED_CONTROL_CHARS, true);
        objectMapper.configure(JsonParser.Feature.INTERN_FIELD_NAMES, true);
        objectMapper.configure(JsonParser.Feature.CANONICALIZE_FIELD_NAMES, true);
        objectMapper.configure(DeserializationConfig.Feature.FAIL_ON_UNKNOWN_PROPERTIES, false);
    }
```

1.**反序列化 DeserializationFeature**

```
FAIL_ON_UNKNOWN_PROPERTIES  	
设置为false，表示：json中字段多了，不会影响json转Object
```

```
ACCEPT_EMPTY_STRING_AS_NULL_OBJECT
设置为ture时，可以将一个空字符串“”转成一个null。如{“student”:””}，其中“student”在反序列化时对应类Student，此时Student的值会被设置为null。
```

2.**序列化 SerializationFeature**

```
WRITE_NULL_MAP_VALUES   如果为false，则表示跳过null的字段
```

3.**常用注解**

```
@JsonIgnore 此注解用于属性上，作用是进行JSON操作时忽略该属性
@JsonProperty 将类成员的名称序列化时，变为另外一个名称。如@JsonProperty(“bank_code”)。
@JsonInclude @JsonInclude(JsonInclude.Include.NON_NULL）表示跳过值为null的类成员
@JsonFormat 对date类型进行格式化，@JsonFormat(pattern = “yyyy-MM-dd HH:mm:ss”)
```

