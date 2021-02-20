# Java中判断对象是否为空的方法

## 工具StringUtils的判断方法：

一种是org.apache.commons.lang3包下的；
另一种是org.springframework.util包下的。这两种StringUtils工具类判断对象是否为空是有差距的：



```java

StringUtils.isEmpty(CharSequence cs); //org.apache.commons.lang3包下的StringUtils类，判断是否为空的方法参数是字符序列类，也就是String类型

StringUtils.isEmpty(Object str); //而org.springframework.util包下的参数是Object类，也就是不仅仅能判断String类型，还能判断其他类型，比如Long等类型。

```



可以看出第二种的StringUtils类更实用。



org.apache.commons.lang3的StringUtils.isEmpty(CharSequence cs)源码：

```java
public static boolean isEmpty(final CharSequence cs) {
        return cs == null || cs.length() == 0;
}
```

org.springframework.util的StringUtils.isEmpty(Object str)源码：

```java
public static boolean isEmpty(Object str) {
        return (str == null || "".equals(str));
}
```



==基本上判断对象是否为空，StringUtils.isEmpty(Object str)这个方法都能搞定。==



- 接下来就是判断数组是否为空

- ```java
  list.isEmpty(); //返回boolean类型。
  ```

  



## 