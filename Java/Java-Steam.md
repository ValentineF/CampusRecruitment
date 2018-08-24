# Steam简介

## Stream的作用

- 由于对于集合进行操作的需求越发增多，一般会使用循环遍历的方式来对集合元素进行操作，比较笨拙且低效.
- Stream是一种辅助集合操作的工具，针对集合的数据元素进行操作，本身并不存储数据，使用简洁且支持并发访问
- 流和它的名字一样，是顺序执行的操作，数据只能遍历一次

## 流的构造

```java
// 1. Individual values
Stream stream = Stream.of("a", "b", "c");
// 2. Collections中的stream()方法转换成流
List<String> list = Arrays.asList(strArray);
stream = list.stream();
// 3. 封装了常见的基本数值类型的流，免去装箱拆箱操作
IntStream intStream = IntStream.of(1,2,3);
```

## 流的转换

```java
// 1. Array
String[] strArray1 = stream.toArray(String[]::new);
// 2. Collection
List<String> list1 = stream.collect(Collectors.toList());
Set set1 = stream.collect(Collectors.toSet());
// 3. String
String str = stream.collect(Collectors.joining()).toString();
```

## 流的操作

源码：java.util.stream

### filter

过滤出符合条件的元素，生成一个新的Stream

```java
Stream<T> filter(Predicate<? super T> predicate);

//找出偶数集合
Integer[] nums = {1, 2, 3, 4, 5, 6};
Integer[] evens =
Stream.of(sixNums).filter(n -> n%2 == 0).
toArray(Integer[]::new);
```

### map/flatMap

- 将元素映射成为另外一个元素（经过自定义操作）

```java
 <R> Stream<R> map(Function<? super T, ? extends R> mapper);

 //转换为平方数
List<Integer> nums = Arrays.asList(1, 2, 3, 4);
List<Integer> squareNums = nums.stream().
map(n -> n * n).
collect(Collectors.toList());
```

- flatMap适用于一对多的场景(元素本身包含多个底层元素)

```java
Stream<List<Integer>> inputStream = Stream.of(
 Arrays.asList(1),
 Arrays.asList(2, 3),
 Arrays.asList(4, 5, 6)
 );
Stream<Integer> outputStream = inputStream.
flatMap((childList) -> childList.stream());

```

- 其他的mapToInt、mapToLong、mapToDouble只是更方便的转换为指定类型的流

```java
 IntStream mapToInt(ToIntFunction<? super T> mapper);
 LongStream mapToLong(ToLongFunction<? super T> mapper);
 DoubleStream mapToDouble(ToDoubleFunction<? super T> mapper);
```

### distinct

对元素进行去重操作（默认使用objct的equals方法）

```java
Stream<T> distinct();
//去重
List<Integer> nums = Arrays.asList(1,1,2,3,3,5,4,2);
nums = nums.stream().distinct().collect(Collectors.toList());
//输出结果：12354
```

### sorted

- 对元素进行排序操作，按照默认的Comparable进行排序
- 如果没有实现Comparable，会抛出 ClassCastException，此时可添加Comparator作为参数

```java
Stream<T> sorted();
Stream<T> sorted(Comparator<? super T> comparator);
//去重&排序
List<Integer> nums = Arrays.asList(1,1,2,3,3,5,4,2);
nums = nums.stream().distinct().sorted().collect(Collectors.toList());
//输出结果：12345
```

### peek

主要是为了debug，能提供每个元素的展示性操作(就是print了)，没啥用

```java
Stream<T> peek(Consumer<? super T> action);
//print...
 Stream.of("one", "two", "three", "four")
    .filter(e -> e.length() > 3)
    .peek(e -> System.out.println("Filtered value: " + e))
    .map(String::toUpperCase)
    .peek(e -> System.out.println("Mapped value: " + e))
    .collect(Collectors.toList());
//输出：three，THREE，four，FOUR
```

### limit

顾名思义，限制返回的元素数目

```java
Stream<T> limit(long maxSize);
//限制数目
List<String> list = Stream.of("one", "two", "three", "four").limit(1).collect(Collectors.toList());
//输出：one
```

### skip

跳过前x个元素后的stream

```java
Stream<T> skip(long n);
//跳过2个
List<Integer> list = Stream.of(1,2,3,4,5).skip(2).collect(Collectors.toList());
//输出：3，4，5
```

### forEach

循环遍历操作，接受一个lambda表达式

```java
void forEach(Consumer<? super T> action);
//打印每个元素
Stream.of("one", "two", "three", "four").forEach(num->System.out.println(num));
//和peek不同，foreach每个元素后，该元素就消耗没了。Peek可用多次使用该元素
```

### reduce

组合元素，将流内的元素组合起来操作

```java
T reduce(T identity, BinaryOperator<T> accumulator);
Optional<T> reduce(BinaryOperator<T> accumulator);
// 字符串连接，concat = "ABCD"
String concat = Stream.of("A", "B", "C", "D").reduce("", String::concat); 
// 求最小值，minValue = -3.0
double minValue = Stream.of(-1.5, 1.0, -3.0, -2.0).reduce(Double.MAX_VALUE, Double::min); 
// 求和，sumValue = 10, 有起始值
int sumValue = Stream.of(1, 2, 3, 4).reduce(0, Integer::sum);
// 求和，sumValue = 10, 无起始值
sumValue = Stream.of(1, 2, 3, 4).reduce(Integer::sum).get();
// 过滤，字符串连接，concat = "ace"
concat = Stream.of("a", "B", "c", "D", "e", "F").
 filter(x -> x.compareTo("Z") > 0).
 reduce("", String::concat);
```

### min/max

找出最值，返回一个optional

```java
Optional<T> min(Comparator<? super T> comparator);
Optional<T> max(Comparator<? super T> comparator);
//返回一个optional
Optional<Integer> min = Stream.of(1, 2, 3, 4).min(new Comparator<Integer>() {
    @Override
    public int compare(Integer o1, Integer o2) {
        return 0;
    }
});
```

### count

计数

```java
long count();
//
long count = Stream.of(1, 2, 3, 4).count();
```

### Match

- 有三种形式：allMatch/anyMatch/noneMatch
- 只要符合条件就返回true

```java
boolean anyMatch(Predicate<? super T> predicate);
//判断是否有偶数
boolean hasEven = Stream.of(1, 2, 3, 4).anyMatch(num->num%2 == 0);
//输出：true
```

## 参考资料

- [Java 8 中的 Streams API 详解](https://www.ibm.com/developerworks/cn/java/j-lo-java8streamapi/index.html)