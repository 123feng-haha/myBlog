# Dozer 时间转换问题
```
使用Dozer的原因 是因为po、dto转为vo 的互转
不用的话 就要一个个set，get
如果每个实体都要配置一个xml 的话开发效率很低
```

实例 场景 （就是这么方便）

```java
     @ResponseBody
    public RtnResult findPageOrderByWhere(@RequestBody FinancingOrderVo vo) {
        FinancingOrder forder=DozerUtils.map(vo, FinancingOrder.class);
    ......................... 
```

/**
*测试
*/

```java

testEntityVo vo = new testEntityVo();
   vo.setTdDate("2019-05-25 09:23:45:00");
    vo.setBytes("1");
    vo.setBmoney("100.11");
    vo.setMun("100");
    vo.setCreaterId("1000");
    vo.setName("dozer");
    // vo 转换成po
    testEntity o = DozerUtils.map(vo, testEntity.class);//转换
    testEntityVo os = DozerUtils.map(o, testEntityVo.class);
    System.out.println(o.toJson());
    System.out.println(os.toJson());
```

打印日志

```java
{"bmoney":100.11,"bytes":1,"createrId":1000,"mun":100,"name":"dozer","tdDate":1558747425000}

{"bmoney":"100.11","bytes":"1","createrId":"1000","mun":"100","name":"dozer","tdDate":"2019-05-25 09:23:45:00"}
```

直接贴代码吧

### jar maven 版
```java
<!-- dozer -->
        <dependency>
            <groupId>net.sf.dozer</groupId>
            <artifactId>dozer</artifactId>
            <version>5.5.1</version>
        </dependency>
```



### DozerUtils.java

```java

package com;

import java.util.ArrayList;
import java.util.Arrays;
import java.util.Collection;
import java.util.List;

import org.dozer.DozerBeanMapper;


/**

 * 
 * @className: DozerUtils
 * @description: DTO/VO/DO等之间的转换
 * *
    */

public class DozerUtils {
    /**
     * 持有Dozer单例, 避免重复创建DozerMapper消耗资源.
     */
    private static DozerBeanMapper dozer;
static {
    if (dozer == null) {
        dozer = new DozerBeanMapper();
         List<String> mappingFileUrls = Arrays.asList("dozer/dozer-date.xml");
         dozer.setMappingFiles(mappingFileUrls);
    }
}
/**
 * 
 * @title: map
 * @description: 单个对象相互转换
 *
 * @param source 源对象
 * @param destinationClass 目标对象
 * @return
 * @date 2017年11月8日 下午6:08:54
 */
public static <T> T map ( Object source, Class<T> destinationClass ) {

    return dozer.map(source, destinationClass);
}

/**
 * 
 * @title: mapList
 * @description: 集合对象相互转换
 *
 * @param sourceList
 * @param destinationClass
 * @return
 * @date 2017年11月8日 下午6:09:41
 */
@SuppressWarnings("rawtypes")
public static <T> List<T> mapList ( Collection sourceList, Class<T> destinationClass ) {
    List<T> destinationList = new ArrayList<T>();
    for (Object sourceObject : sourceList) {
        T destinationObject = dozer.map(sourceObject, destinationClass);
        destinationList.add(destinationObject);
    }

    return destinationList;
}
```

}

### dozer-date.xml

```java
<?xml version="1.0" encoding="UTF-8"?>
<mappings xmlns="http://dozer.sourceforge.net"
          xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
          xsi:schemaLocation="http://dozer.sourceforge.net
          http://dozer.sourceforge.net/schema/beanmapping.xsd">
  <configuration>
    <custom-converters> <!-- these are always bi-directional -->
      <converter type="com.gls.test.StringToDateConverter" >
        <class-a>java.lang.String</class-a>
        <class-b>java.util.Date</class-b>
      </converter>
    </custom-converters>     
  </configuration>

</mappings>

```

StringToDateConverter.java 这个路径要配置在dozer-dadte.xml 中

```java
package com.gls.test;

import java.text.ParsePosition;
import java.text.SimpleDateFormat;
import java.util.Date;

import org.dozer.DozerConverter;

public class StringToDateConverter extends DozerConverter<String, Date> {
public StringToDateConverter() {
    super(String.class, Date.class);
}

@Override
public String convertFrom(Date source, String destination) {
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SS");
    destination = formatter.format(source);
    return destination;
}

@Override
public Date convertTo(String source, Date destination) {
    SimpleDateFormat formatter = new SimpleDateFormat("yyyy-MM-dd HH:mm:ss:SS");
    ParsePosition pos = new ParsePosition(0);
    destination = formatter.parse(source, pos);
    return destination;
}
```

