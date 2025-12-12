# 接口的默认方法
- 通过defualt关键字实现接口方法的默认实现
```java
interface Formula{

    double calculate(int a);
   //默认实现
    default double sqrt(int a) {
        return Math.sqrt(a);
    }

}
public class Main {
  public static void main(String[] args) {
    // 通过匿名内部类方式访问接口
    Formula formula = new Formula() {
        @Override
        public double calculate(int a) {
            //不用写具体实现即可调用
            return sqrt(a * 100);
        }
    };

    System.out.println(formula.calculate(100));     // 100.0
    System.out.println(formula.sqrt(16));           // 4.0

  }

}
```
# 函数式接口
- 只包含一个抽象方法的接口，可以有多个default/static/private修饰的方法
- 接口中的非default/static/private有方法本来就是 public abstract即抽象方法
- default/static/private修饰的方法必须实现，有方法体
- @FunctionalInterface校验是否是函数式接口

# 方法引用
- 需要把“某个方法”传给一个函数式接口，该方法作为该接口抽象方法的实现
- 类名：：方法
## 静态方法引用
```java
//抽象接口，传入形参与返回结果一致
//都是传String返回Integer
 Converter<String, Integer> converter = Integer::valueOf;
```
## 未绑定对象方法引用
- 类名是第一个形参，调用的是第一个形参的方法
```java

Predicate<String> p = String::isEmpty;
// 等价于 s -> s.isEmpty()

BiFunction<String, String, Integer> cmp = String::compareTo;
// 等价于 (a, b) -> a.compareTo(b)

```
## 绑定对象方法引用
```java
class Something {
    String startsWith(String s) {
        return String.valueOf(s.charAt(0));
    }
}
Something something = new Something();
Converter<String, String> converter = something::startsWith;
String converted = converter.convert("Java");
System.out.println(converted);    // "J"
```
## 构造器引用
```java
//定义构造器
class Person {
    String firstName;
    String lastName;

    Person() {}

    Person(String firstName, String lastName) {
        this.firstName = firstName;
        this.lastName = lastName;
    }
}
//对象工厂接口接口
interface PersonFactory<P extends Person> {
    P create(String firstName, String lastName);
}
//构造函数引用
PersonFactory<Person> personFactory = Person::new;
Person person = personFactory.create("Peter", "Parker");

```
# Optional
https://blog.kaaass.net/archives/764<br>
- 防止 NullPointerException
```java
//of()：为非null的值创建一个Optional
Optional<String> optional = Optional.of("bam");
// isPresent()：如果值存在返回true，否则返回false
optional.isPresent();           // true
//get()：如果Optional有值则将其返回，否则抛出NoSuchElementException
optional.get();                 // "bam"
//orElse()：如果有值则将其返回，否则返回指定的其它值
optional.orElse("fallback");    // "bam"
//ifPresent()：如果Optional实例有值则为其调用consumer，否则不做处理
optional.ifPresent((s) -> System.out.println(s.charAt(0)));     // "b"
```

# Stream流
## 创建流
```java
// 来自集合（最常用）
List<String> list = Arrays.asList("a", "b", "c");
Stream<String> s1 = list.stream();        // 顺序流
Stream<String> s2 = list.parallelStream(); // 并行流（谨慎使用）

// 来自数组
String[] arr = {"x", "y", "z"};
Stream<String> s3 = Arrays.stream(arr); // 或 Arrays.stream(arr, start, end)

// 直接创建
Stream<String> s4 = Stream.of("A", "B", "C");
```
## 操作
- 中间操作：Filter(过滤)，Sorted(排序)，Map(映射)
- 最终操作：Match(匹配),Count(计数),Reduce(规约),forEach(迭代)
### forEach
-  'forEach' 来迭代流中的每个数据,以下代码片段使用 forEach 输出了10个随机数
```java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```
### map
- 'map' 来映射每个元素到对应的结果，以下代码片段使用 map 输出了元素对应的平方数
```java
List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);
// 获取对应的平方数
List<Integer> squaresList = numbers.stream().map( i -> i*i).distinct().collect(Collectors.toList());
```
### filter
- 'filter' 通过设置的条件过滤出元素,以下代码片段使用 filter 方法过滤出空字符串
```java
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
// 获取空字符串的数量
long count = strings.stream().filter(string -> string.isEmpty()).count();
```
### limit
- 'limit' 方法用于获取指定数量的流,以下代码片段使用 limit 方法打印出 10 条数据
```java
Random random = new Random();
random.ints().limit(10).forEach(System.out::println);
```
### sort
- 'sort' 用于对流进行排序。以下代码片段使用 sorted 方法对输出的 10 个随机数进行排序
```java
Random random = new Random();
random.ints().limit(10).sorted().forEach(System.out::println);
```
### reduce
- 'reduce' 用于归约
```java
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
### count
- 'count' 用于计数
```java
        long count = words.stream().count(); // 终止：计数
        System.out.println("单词数量：" + count);
```
### toArray
- 'toArray'用于转数组
```java
        // 终止：转数组
        String[] arr = words.stream().toArray(String[]::new);
        System.out.println("数组：" + Arrays.toString(arr));
```
### 并行（parallelStream()）程序
- 顺序流：单线程、顺序处理、开销低、适合大多数一般场景。
- 并行流：多线程、可能乱序、需要无副作用与易拆分的数据源，在合适场景下能提升性能；否则反而更慢
### Collectors
- 'Collectors' 可用于返回list或字符串：
```java
//结果为list集合
List<String>strings = Arrays.asList("abc", "", "bc", "efg", "abcd","", "jkl");
List<String> filtered = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.toList());
 
//结果为字符串
System.out.println("筛选列表: " + filtered);
String mergedString = strings.stream().filter(string -> !string.isEmpty()).collect(Collectors.joining(", "));
System.out.println("合并字符串: " + mergedString);
```
### 统计
- summaryStatistics 先用mapToInt转换成对应类型，然后生成带有一系列方法的对象
```java

List<Integer> numbers = Arrays.asList(3, 2, 2, 3, 7, 3, 5);

IntSummaryStatistics stats = numbers.stream()
    .mapToInt(x -> x)             // 转为 IntStream（避免装箱/拆箱）
    .summaryStatistics();          // 计算汇总统计

System.out.println("列表中最大的数 : " + stats.getMax());
System.out.println("列表中最小的数 : " + stats.getMin());
System.out.println("所有数之和 : " + stats.getSum());
System.out.println("平均数 : " + stats.getAverage());

//Long/Double 与通用 Collectors
LongSummaryStatistics longStats = longList.stream()
    .mapToLong(x -> x)
    .summaryStatistics();

DoubleSummaryStatistics doubleStats = doubleList.stream()
    .mapToDouble(x -> x)
    .summaryStatistics();

// 或者使用收集器：
LongSummaryStatistics longStats2 = longList.stream()
    .collect(Collectors.summarizingLong(Long::longValue));

DoubleSummaryStatistics doubleStats2 = doubleList.stream()
    .collect(Collectors.summarizingDouble(Double::doubleValue));

```
# 日期API
## 本地化日期时间 API
- LocalDate/LocalTime 和 LocalDateTime 类
```java
import java.time.LocalDate;
import java.time.LocalTime;
import java.time.LocalDateTime;
import java.time.Month;
 
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester java8tester = new Java8Tester();
      java8tester.testLocalDateTime();
   }
    
   public void testLocalDateTime(){
    
      // 获取当前的日期时间
      LocalDateTime currentTime = LocalDateTime.now();
      System.out.println("当前时间: " + currentTime);  //当前时间: 2016-04-15T16:55:48.668
        
      LocalDate date1 = currentTime.toLocalDate();
      System.out.println("date1: " + date1);     //date1: 2016-04-15
        
      Month month = currentTime.getMonth();
      int day = currentTime.getDayOfMonth();
      int seconds = currentTime.getSecond();
        
      System.out.println("月: " + month +", 日: " + day +", 秒: " + seconds);  //月: APRIL, 日: 15, 秒: 48
        
      LocalDateTime date2 = currentTime.withDayOfMonth(10).withYear(2012);
      System.out.println("date2: " + date2);  //date2: 2012-04-10T16:55:48.668
        
      // 12 december 2014
      LocalDate date3 = LocalDate.of(2014, Month.DECEMBER, 12);
      System.out.println("date3: " + date3);   //date3: 2014-12-12
        
      // 22 小时 15 分钟
      LocalTime date4 = LocalTime.of(22, 15);
      System.out.println("date4: " + date4);   //date4: 22:15
        
      // 解析字符串
      LocalTime date5 = LocalTime.parse("20:15:30");
      System.out.println("date5: " + date5);    //date5: 20:15:30
   }
}
```
## 使用时区的日期时间API
```java
import java.time.ZonedDateTime;
import java.time.ZoneId;
 
public class Java8Tester {
   public static void main(String args[]){
      Java8Tester java8tester = new Java8Tester();
      java8tester.testZonedDateTime();
   }
    
   public void testZonedDateTime(){
    
      // 获取当前时间日期
      ZonedDateTime date1 = ZonedDateTime.parse("2015-12-03T10:15:30+05:30[Asia/Shanghai]");
      System.out.println("date1: " + date1);  //date1: 2015-12-03T10:15:30+08:00[Asia/Shanghai]
        
      ZoneId id = ZoneId.of("Europe/Paris");
      System.out.println("ZoneId: " + id);  //ZoneId: Europe/Paris
        
      ZoneId currentZone = ZoneId.systemDefault();
      System.out.println("当期时区: " + currentZone);  //当期时区: Asia/Shanghai
   }
}
```
