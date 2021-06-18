# 一文搞懂 Java 泛型

## 前言

最近在网上看到很多新手不太理解 Java 中的泛型，尤其是对于源码中各种通配符 "?"、"T"、"S"、"R" 等，不理解其含义，更不知如何使用泛型。本篇文章将从头开始透彻的分析 Java 中的泛型，并结合项目实际应用场景，希望对初学者有帮助。

## 什么是泛型&为什么引入泛型

在谈泛型之前，我们先来看一段 JDK5 之前没有泛型时的代码

```
    public static void main(String[] args) {
        ArrayList list = new ArrayList();
        list.add(521);//添加 Integer 类型元素
        list.add("wan");//添加 String 类型元素
        list.add(true);//添加 Boolean 类型元素
        list.add('a');//添加 Character 类型元素
        Object item1 = list.get(0);//只能用 Object 接受元素
        list.forEach(item -> {
            //使用 item，这里的 item 类型是 Object，由于不知道 item 的确切类型，我们需要判断之后强转
            if (item instanceof Integer) {
                //执行业务...
            } else if (item instanceof String) {
                //执行业务...
            } else if (item instanceof Boolean) {
                //执行业务...
            } //继续判断类型...
        });
    }
复制代码
```

没有泛型的时候，我们声明的 List 集合默认是可以存储任意类型元素的，乍一看你可能还会觉得挺好，这样功能强大，啥类型都可以存储......但是开发的时候由于不知道集合中元素的确切类型，遍历的时候我们拿到的 item 其实是 Object 类型，如果要使用就必须强转，强转就必须得判断当前元素的具体类型，否则直接使用强转很可能会发生类型转换异常。这样就会让开发很不方便，每次都要额外做判断工作。

**总结起来就是一句话，它不安全！**

那么你可能已经想到了，我们在业务中不要把全部数据都存放在一个 List 就行了，在代码中定义多个 List 分类型使用

```
    public static void main(String[] args) {
        ArrayList listInteger = new ArrayList();
        ArrayList listString = new ArrayList();
        ArrayList listBoolean = new ArrayList();
        //...这样就可以在不同的 list 中存入对应的类型数据
        //——————————————————————分割线————————————————————
        listString.add(121);//即使如此它还是无法限制，只能起到提示作用
    }
复制代码
```

你看上面的代码其实治标不治本，我们声明了 listString 是想让它只存储 String 类型，但是我们仍然可以存储非 String 类型的数据，而且更为重要的是这种类型转换异常通常只有在运行时才会被发现。我们需要一种机制能强制性的让我们只能存储对应类型的元素，否则编译就不通过，所以泛型出现了。

事实上泛型也是我们刚刚的思路，在实例化时给集合分配一个类型，限定一个集合只能存储我们分配的类型的元素

```
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("wan");
        list.add(521);//编译报错，只能存储 String 元素
        String str = list.get(0);//直接用 String 类型接受元素
        list.forEach(item -> {
            //这里 item 类型就是 String
        });
    }
复制代码
```

有了泛型的指定，我们声明的 list 就只能存储规定类型 String ，当存储其他类型的元素时编辑器就会直接给我们报错（可以在 IDEA 开发环境中看 add 方法提示参数类型就是 String），这样类型不匹配的问题就在编译时候就能检查出来，而不会在运行时才抛出异常。而且当我们进行遍历、获取元素等操作时，get 方法返回值就是 String 类型的

### 总结泛型的好处

- **编译期类型安全**
- **避免了强制类型转换运行时异常**
- **同一个类可以操作多种类型数据，代码复用**

其实到这里我还没有给泛型下定义，别急，看完下一节 **泛型类**

## 泛型类

说到泛型类，最典型的例子就是上面我们说的 ArrayList 了。你有没有想过，为什么给 ArrayList 指定泛型之后，就只能存储指定类型，get(0) 获取元素返回值就是指定的那个泛型类型？看下 ArrayList 部分源码

```
    //类定义
    public class ArrayList<E> extends AbstractList<E> implements List<E>
    
    //添加元素方法
    public boolean add(E e) {...}
    
    //获取元素方法
    public E get(int index) {...}
复制代码
```

可以看到 ArrayList 在定义的时候指定了一个泛型 <E>，并且下面的添加元素、获取元素等方法也都是对这个 E 进行操作，我相信初学者在看到这个的时候肯定是懵逼的......这个 E 是什么鬼？其实这个 E 就是我们实例化 ArrayList 时指定的类型，当我们指定 String，add 方法的形参和 get 方法的返回值就是 String 类型，当我们指定 Integer，add 方法的形参和 get 方法的返回值就是 Integer 类型。这样一来，ArrayList 这个类就被参数化了，当实例化 ArrayList 时传入不同的泛型就可以操作不同的类型。

当然我们也可以在一个类中定义多个泛型参数，比如 HashMap

```
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>
复制代码
```

**定义：泛型的本质就是把类型参数化，所操作的数据类型被指定为参数，根据动态传入进行处理**

### 自定义泛型类

上面我们看到的是 JDK 源码中泛型类，当然我们自己也可以定义泛型类，最常见的就是我们曾经封装的 web 应用后端返回结果。

```
    public class ResultHelper<T> implements Serializable {
        private T data;
        private boolean success;
        private Integer code;
        private String message;

        private ResultHelper() {}

        public static <T> ResultHelper<T> wrapSuccessfulResult(T data) {
            ResultHelper<T> result = new ResultHelper<T>();
            result.data = data;
            result.success = true;
            result.code = 0;
            return result;
        }
    }
复制代码
```

这是很多公司会用的工具类，封装一个这样的数据结构给前端（不过现在大部分企业已经不这么用了）。这样一来，我的 wrapSuccessfulResult 泛型方法的参数泛型 T 可以接收调用者任何参数，无论是订单数据还是商品数据等等，都可以封装。

## 泛型方法

有时候开发中我们会有这样一种需求，根据方法传入的参数类型不同，返回不同的返回值类型。上面所说的自定义泛型类 wrapSuccessfulResult 方法就是典型的泛型方法，它只有一个泛型参数，我们还可以使用多个泛型参数：

```
   public static <E, T> List<T> convertListToList(List<E> input, Class<T> clzz) {
        List<T> output = Lists.newArrayList();
        if (CollectionUtils.isNotEmpty(input)) {
            input.forEach(source -> {
                T target = BeanUtils.instantiate(clzz);
                BeanUtils.copyProperties(source, target);
                output.add(target);
            });
        }
        return output;
    }
复制代码
```

例如上面这个工具方法（不用太注重方法体），它的作用就是把一个类型 E 的 List 转换为另一个类型 T 的 List。那这里 E 和 T 都是开放的，随便调用者传递什么类型。

## 无界泛型通配符 "?"

"?" 代表不确定的类型，比如我们公司 APP 有一个订单列表/详情的需求，我们都知道订单有待发货、待收货、已退款等不同页面，不同状态页面的数据又不一样。如果把所有字段都放在一个类中，那样的设计太难受了，代码看起来更难受（实际上第一版就是这么干的，后来是我改的）。比如待收货有发货时间，待支付就没有，如果你用一个有发货时间字段的类来作为待支付详情的数据结构，那就不合适。所以我写了一个父类 AppOrderResponse 把所有页面共有的字段（订单id、订单编号、订单状态、下单时间等）放在父类，其他独有的再扩展子类，继承关系为：

![image.png](https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bc790070d5684dc690c2d0b65fe6927a~tplv-k3u1fbpfcp-watermark.image)

这样根据不同状态返回对应的类型数据就行了，比如待发货列表就返回 AppOrderWaitDeliveredResponse 泛型类型，待收货列表就返回 AppOrderDispatchedResponse 泛型类型，不过我们一个接口要同时返回四种可能的类型，这该怎么办呢？也许你可能会这么想

```
  public IPage<AppOrderResponse> list(Page<AppOrderResponse> page, AppOrderQueryRequest request) {
        IPage<AppOrderDispatchedResponse> demo1 = new Page<>();
        IPage<AppOrderWaitDeliveredResponse> demo2 = new Page<>();
        //return demo1;//报错
        return demo2;//报错
    }
复制代码
```

将返回值定义为 IPage<AppOrderResponse> 类型，这样我就可以返回各种类型了，但其实这样会报错。AppOrderResponse 是 AppOrderWaitDeliveredResponse 的父类，但是 IPage<AppOrderResponse> 并不是 IPage<AppOrderWaitDeliveredResponse> 的父类，泛型中的继承不是你想的那样。此时就需要使用通配符 "?" 来解决。

```
    public IPage<?> list(Page<AppOrderResponse> page, AppOrderQueryRequest request) {
        return appOrderService.list(page, request);
    }
复制代码
```

这样一来我们使用通配符 "?" 之后返回任何 IPage 泛型都可以。但是这显然不合适，因为我们的类型是 AppOrderResponse ，不可能允许程序返回一个不属于 AppOrderResponse 的泛型类型。所以我们可以使用**泛型的上界下界**来控制

## 泛型的上界下界

正如上面订单列表的例子，我们应该限定返回值的泛型仅为 AppOrderResponse 或者其子类，可以这么写

```
    public IPage<? extends AppOrderResponse> list(Page<AppOrderResponse> page, AppOrderQueryRequest request) {
        return appOrderService.list(page, request);
    }
复制代码
```

这种写法叫做指定泛型的上界（上限），我们不能直接用 IPage<AppOrderResponse> 表示上界，但是可以使用 IPage<? extends AppOrderResponse>

其实这从 extends 和 super 关键字很容易理解

```
    IPage<? extends AppOrderResponse> //表示泛型最高类型是 AppOrderResponse，只能是它或它的子类
    IPage<? super AppOrderResponse> //表示泛型最低类型是 AppOrderResponse。只能是它或它的父类
复制代码
```

上面我们都是把无界通配符用在返回值，当然无界通配符也是可以用在方法参数上的

### 使用上界参数

```
    public void test1(List<? extends AppOrderResponse> list){
        list.add(new AppOrderDispatchedResponse());//添加元素报错，因为我们传入的可能是 List<AppOrderWaitPaidResponse>
        list.add(new AppOrderResponse());//添加元素报错,因为我们传入的可能是 List<AppOrderWaitPaidResponse>
        list.add(null);//这是唯一可以添加的元素 null
        AppOrderResponse appOrderResponse = list.get(0);//接受元素类型为 AppOrderResponse
    }
复制代码
```

当使用上界参数时，不可以对参数进行新增元素。因为参数需要的泛型是 AppOrderResponse 及其子类，我们在方法体中添加元素会报错是因为传入进来的有可能是 AppOrderResponse，也有可能是它的子类集合。比如我们传入的参数是 List<AppOrderDispatchedResponse> 那么这个 list 能够添加的元素就只能是 AppOrderDispatchedResponse ，而不能添加它的同级别类或者父类，所以当不能涵盖所有场景的时候，程序就不允许我们添加。有意思的是它可以添加 null ......

而当我们获取元素时，这个元素一定是 AppOrderResponse 或者其子类类型的，所以我们可以用 AppOrderResponse 来接受，用父类引用指向子类对象。

**泛型上界参数在方法内只能读取，不能写入**

### 使用下界参数

```
    public void test2(List<? super AppOrderResponse> list){
        list.add(new AppOrderResponse());//可以添加
        list.add(new AppOrderDispatchedResponse());//可以添加
        Object object = list.get(0);//接受元素的类型为 Object
    }
复制代码
```

使用下界参数时，由于参数需要的泛型是 AppOrderResponse 及其父类，那么我们添加 AppOrderDispatchedResponse 类型的元素当然是可以的，因为 AppOrderDispatchedResponse 是子类，肯定是 AppOrderResponse 的类型，这也就是父类引用指向子类对象。

而当我们获取元素时，由于 list 中的元素类型是 AppOrderResponse 父类类型，所以我们必须得用 Object 来接受，这样一来其实没有意义，我拿一个 Object 干嘛呢。

**泛型下界参数在方法内只能写入，不能读取**

## 泛型通配符 "?" 和 T、E、R、K、V 的区别

我相信这是广大同学最容易混淆的地方，毕竟源码中到处都是这些通配符，也看不出有什么区别。其实 T、E、R、K、V 对于程序运行没有区别，定义泛型的时候用 A-Z 中任何一个字母都可以，只不过我们上面的几个是约定俗成的，也算一种规范。

- **T、E、R、K、V 对于程序运行时没有区别，只是名字不同**
- **? 表示不确定的泛型类型**
- **T (type) 表示具体的一个泛型类型**
- **K V (key value) 分别代表 Map 中的键值 Key Value**
- **E (element) 代表元素，例如 ArrayList 中的元素**

那无界通配符 "?" 和它们有啥区别呢？

- **T 用于定义泛型类和泛型方法**

比如我们上面 **泛型类** 的代码示例，用 T 来定义一个泛型，并且可以在代码中对 T 进行操作。而 T 不可以单独作为方法形参，只能在定义的泛型类中或者定义泛型方法才能作为方法形参。

```
    public class ResultHelper<T> implements Serializable {
    private T data;
    ...
    data.toString();
    data.equals(obj);
    ...
    public static <T> ResultHelper<T> wrapSuccessfulResult(T data) {}
复制代码
```

- **"?" 用于方法形参**

比如我们在 **无界通配符 "?"** 的代码示例，即使不在泛型类中，"?" 也可以作为方法形参，定义方法返回值等，但是我们不能对 "?" 进行单独定义、操作

```
    private List<?> list;
    private ? data;//报错
    public class OrderRequest<?> {}//报错
    public void test(? item){}//报错
复制代码
```

## 泛型擦除

所谓的泛型擦除其实很简单，简单来说就是泛型只在编译时起作用，运行时泛型还是被当成 Object 来处理，示例代码

```
    public static void main(String[] args) {
        ArrayList<String> list = new ArrayList<>();
        list.add("wan");//add 方法形参类型为 String
        String s = list.get(0);//get方法返回值类型为 String
        ArrayList<Integer> list2 = new ArrayList<>();
        System.out.println("list 和 list2 类型相同吗：" + (list.getClass() == list2.getClass()));//true 两个 ArrayList 是同一个类型的
        Method[] methods = list.getClass().getMethods();
        for (Method method : methods) {
            method.setAccessible(true);
            if (method.getName().equals("add")) {
                Class<?>[] parameterTypes = method.getParameterTypes();
                if (parameterTypes.length == 1) {
                    for (Class<?> parameterType : parameterTypes) {
                        System.out.println("add(E e) 形参 E 的类型为：" + parameterType.getName());//泛型的参数 E 运行时是 Object 类型
                    }
                }
            } else if (method.getName().equals("get")) {
                Class<?> returnType = method.getReturnType();
                System.out.println("E get(int index) 的返回值 E 的类型为：" + returnType.getName());
            }
        }
    }
复制代码
```

打印结果

```
list 和 list2 在运行时类型相同吗：true
add(E e) 形参 E 在运行时的类型为：java.lang.Object
E get(int index) 的返回值 E 在运行时的类型为：java.lang.Object
复制代码
```

可以看到我们实例化 ArrayList 时虽然传入不同的泛型，但其实它们仍然还是同一个类型。对于 add 方法的形参和 get 方法的返回值，按道理说我们指定的泛型是 String 那么打印出来应该是 String 才对，但是这里运行时得到的却都是 Object，所以这就足以证明了，泛型在编译期起作用，运行时一律被擦除当做 Object 看待，这就是泛型擦除。

## 结语

泛型这个东西理解起来其实真的很简单，难的是如何把它用好，这个需要很强的编程功底、设计模式，我的建议是多看 JDK 源码、框架源码，看大牛是如何在框架中使用的。