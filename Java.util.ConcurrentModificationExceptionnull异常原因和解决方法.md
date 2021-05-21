

# Java.util.ConcurrentModificationException:null异常原因和解决方法

最近在做map的循环遍历时，出现 java.util.ConcurrentModificationException的异常。

分析：

迭代器的线程和集合的线程不是同步的，最开始迭代器计算出了集合的size,当集合Size由于增上改查导致size的变化，迭代器并没有同步，这样虚拟机在运行的时候就导致了冲突。

异常原因



```java
//        for(Map.Entry<String, Object> entry : map.entrySet()){
        Iterator<Map.Entry<String, Object>> entries = map.entrySet().iterator();
        while(entries.hasNext()){
            Map.Entry<String, Object> entry = entries.next();
            kvs.forEach(kv -> {
                if (entry.getKey().equals(kv.getSourceKey())) {
                    map.put(kv.getTargetKey(),entry.getValue());
//                    map.remove(entry.getKey());
                } 
            });
        }


    @Data
    @AllArgsConstructor
    @NoArgsConstructor
    static class KV {
        /**
         * 配置的字段key
         */
        private String sourceKey;
        /**
         * db的字段key
         */
        private String targetKey;
    }
```



看下Iterator next方法实现源码

```java
		... ...
		@SuppressWarnings("unchecked")
        public E next() {
            checkForComodification();
            int i = cursor;
            if (i >= size)
                throw new NoSuchElementException();
            Object[] elementData = ArrayList.this.elementData;
            if (i >= elementData.length)
                throw new ConcurrentModificationException();
            cursor = i + 1;
            return (E) elementData[lastRet = i];
        }
		... ...
        final void checkForComodification() {
            if (modCount != expectedModCount)
                throw new ConcurrentModificationException();
        }

```

在next方法中首先调用了checkForComodification方法，该方法会判断modCount是否等于expectedModCount，不等于就会抛出java.util.ConcurrentModificationExcepiton异常。

我们接下来跟踪看一下modCount和expectedModCount的赋值和修改。

modCount是ArrayList的一个属性，继承自抽象类AbstractList，用于表示ArrayList对象被修改次数。

```java
protected transient int modCount = 0;
```

整个ArrayList中修改modCount的方法比较多，有add、remove、clear、ensureCapacityInternal等，凡是设计到ArrayList对象修改的都会自增modCount属性。

在创建Iterator的时候会将modCount赋值给expectedModCount，在遍历ArrayList过程中，没有其他地方可以设置expectedModCount了，因此遍历过程中expectedModCount会一直保持初始值--list的长度(n)（调用add方法添加了n个元素，修改了n次）。

```java
int expectedModCount = modCount; // 创建对象时初始化
```

遍历的时候是不会触发modCount自增的，但是遍历到integer.intValue() ==（特定值）的时候，执行了一次arrayList.remove(Object)，这行代码执行后modCount++变为了n+1，但此时的expectedModCount仍然为n。

```java
1         final void checkForComodification() {
2             if (modCount != expectedModCount)
3                 throw new ConcurrentModificationException();
4         }
```

在执行next方法时，遇到modCount != expectedModCount方法，导致抛出异常java.util.ConcurrentModificationException。

明白了抛出异常的过程，但是为什么要这么做呢？很明显这么做是为了阻止程序员在不允许修改的时候修改对象，起到保护作用，避免出现未知异常。引用网上的一段解释，[点击查看解释来源](http://lz12366.iteye.com/blog/675016)

```
Iterator 是工作在一个独立的线程中，并且拥有一个 mutex 锁。 
Iterator 被创建之后会建立一个指向原来对象的单链索引表，当原来的对象数量发生变化时，这个索引表的内容不会同步改变。
当索引指针往后移动的时候就找不到要迭代的对象，所以按照 fail-fast 原则 Iterator 会马上抛出 java.util.ConcurrentModificationException 异常。
所以 Iterator 在工作的时候是不允许被迭代的对象被改变的。但你可以使用 Iterator 本身的方法 remove() 来删除对象， Iterator.remove() 方法会在删除当前迭代对象的同时维护索引的一致性。
```



## 复现方法2

```
16         // 复现方法二
17         iterator = arrayList.iterator();
18         for (Integer value : arrayList) {
19             Integer integer = iterator.next();
20             if (integer.intValue() == 5) {
21                 arrayList.remove(integer);
22             }
23         }
```

for循环抛异常的原因：

```java
public void forEach(Consumer<? super E> action) {
        Objects.requireNonNull(action);
        final int expectedModCount = modCount;
        @SuppressWarnings("unchecked")
        final E[] elementData = (E[]) this.elementData;
        final int size = this.size;
        for (int i=0; modCount == expectedModCount && i < size; i++) {
            action.accept(elementData[i]);
        }
        if (modCount != expectedModCount) {
            throw new ConcurrentModificationException();
        }
    }
```

在for循环中一开始也是对expectedModCount采用modCount进行赋值。在进行for循环时每次都会有判定条件modCount == expectedModCount，当执行完arrayList.remove(integer)之后，该判定条件返回false退出循环，然后执行if语句，结果同样抛出java.util.ConcurrentModificationException异常。

这两种复现方法实际上都是同一个原因导致的。



## 解决方案

```java
public void test2() {
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            arrayList.add(Integer.valueOf(i));
        }

        Iterator<Integer> iterator = arrayList.iterator();
        while (iterator.hasNext()) {
            Integer integer = iterator.next();
            if (integer.intValue() == 5) {
                iterator.remove();
            }
        }
    }
```

这种解决方案最核心的就是调用iterator.remove()方法。我们看看该方法源码为什么这个方法能避免抛出异常

```java
public void remove() {
            if (lastRet < 0)
                throw new IllegalStateException();
            checkForComodification();

            try {
                ArrayList.this.remove(lastRet);
                cursor = lastRet;
                lastRet = -1;
                expectedModCount = modCount;
            } catch (IndexOutOfBoundsException ex) {
                throw new ConcurrentModificationException();
            }
        }
```

在iterator.remove()方法中，同样调用了ArrayList自身的remove方法，但是调用完之后并非就return了，而是expectedModCount = modCount重置了expectedModCount值，使二者的值继续保持相等。

针对forEach循环并没有修复方案，因此在遍历过程中同时需要修改ArrayList对象，则需要采用iterator遍历。

上面提出的解决方案调用的是iterator.remove()方法，如果不仅仅是想调用remove方法移除元素，还想增加元素，或者替换元素，是否可以呢？浏览Iterator源码可以发现这是不行的，Iterator只提供了remove方法。

但是ArrayList实现了ListIterator接口，ListIterator类继承了Iter，这些操作都是可以实现的，使用示例如下：

```java
public void test3() {
        ArrayList<Integer> arrayList = new ArrayList<>();
        for (int i = 0; i < 20; i++) {
            arrayList.add(Integer.valueOf(i));
        }

        ListIterator<Integer> iterator = arrayList.listIterator();
        while (iterator.hasNext()) {
            Integer integer = iterator.next();
            if (integer.intValue() == 5) {
                iterator.set(Integer.valueOf(6));
                iterator.remove();
                iterator.add(integer);
            }
        }
    }
```

