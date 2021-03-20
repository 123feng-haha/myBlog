### java lambda表达式常用场景及举例

一、现在绝大部分web项目组jdk都升级到了1.8，那么1.8开始引入的lambda表达式确实也带了很大便利，今天将总结lambda表达式的常用示例。

Student student1 = new Student().setId(1L).setName("张三").setCore(60).setGender(1).setTeacherId(2L);

Student student2 = new Student().setId(2L).setName("ben").setCore(90).setGender(2).setTeacherId(3L);

Student student3 = new Student().setId(3L).setName("anny").setCore(100).setGender(2).setTeacherId(2L);

 

1、stream转换为集合，我们可以不再一个一个的add了。

```java
List<Student> students = Stream.of(student1, student2, student3).collect(Collectors.toList());
```

 

2、map对象转换，我们一般从数据库查数据为PO对象，返回给web端时一般为VO对象，那么这其中就涉及对象转换返回，map就提供了便利的操作。

```java
List<StudentVo> studentVos = students.stream().map(student -> {
    StudentVo studentVo = new StudentVo();
    BeanUtils.copyProperties(student, studentVo);
    return studentVo;
}).collect(Collectors.toList());
```

  这样就很方便的转成另一个对象。

 

3、peek用法，用法同map一样，不同的是不需要return返回值。

```java
List<Student> studentVos = students.stream().peek(student -> student.setDeleted(false)).collect(Collectors.toList());
```

  我们可能只是为了修改对象的属性，我们无须使用map。

 

4、filter 过滤器，可以为我们过滤掉符合条件的数据

```java
List<Student> filters = students.stream().filter(Student::getDeleted).collect(Collectors.toList());
```

  这样，我们就过滤掉了被删除的学生，返回未被删除的学生集合。

 

5、sort排序，我们经常会遇到按某个属性值排序的需求。

```java
List<Student> collect = students.stream().sorted(Comparator.comparing(Student::getCore).reversed())
        .sorted(Comparator.comparing(Student::getGender)).collect(Collectors.toList());
```

这是一个连续排序的例子，按学生分数倒叙，分数相同按照性别排序，reversed()表示倒叙，是不是感觉完全实现了sql的功能。

 

6、sort排序扩展，我们经常会遇到按照指定集合排序。比如我们有一个排好序的学生id集合，返回的数据也需要按照给定集合返回

```java
List<Long> webIds = new ArrayList<Long>(){{
    add(2L); add(1L); add(3L);
}};
List<Student> webSortList = students.stream().sorted(Comparator.comparing(student -> {
    int index = webIds.indexOf(student.getId());
    if (index != -1) {
        return index - Long.MAX_VALUE;
    }
    return (long) index;
}, Long::compareTo)).collect(Collectors.toList());
```

我们不需要去写一堆的循环程序，提供了很大的便利。

 

7、min、max等，求最小，最大，平均、求和等是一样的使用方法，这里就一起举例。

```java
int min = students.stream().mapToInt(Student::getCore).min().orElse(-1);
int max = students.stream().mapToInt(Student::getCore).max().orElse(-1);
int sum = students.stream().mapToInt(Student::getCore).sum();
double average = students.stream().mapToInt(Student::getCore).average().orElse(-1);
```



8、分组，既然有聚合，那就有分组，比如我们需要按照性别进行分组。

```java
Map<Integer, List<Student>> genderGroup 
        = students.stream().collect(Collectors.groupingBy(Student::getGender, Collectors.toList()));
```

当然，分组里边还可以再分组。

 

9、Collectors.toMap()常用方式，比如我们需要形成姓名和分数的一个json

```java
Map<String, Integer> nameScore = students.stream().collect(Collectors.toMap(Student::getName, Student::getCore));
```

这里只是举例，不考虑姓名重复问题。

如果要是行id对应自身对象呢？

```java
Map<String, Student> nameStudent = students.stream().collect(Collectors.toMap(Student::getName, Function.identity()));
```

10、list初始化成map，有时我们未了避免空指针问题，会提前初始化一个map

```java
Map<Long, List<Student>> init = webIds.stream().collect(Collectors.toMap(el -> el, el -> new ArrayList<>()));
```

9和10都是一些比较常用的list转map的用法。

 
