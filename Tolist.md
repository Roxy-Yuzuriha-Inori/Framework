1. @TableField的用法
```java
 /*
 value=""  对应表中列名
 exist = false  表明该字段不在表中
 fill = FieldFill.INSERT  自动填充，需要实现 MetaObjectHandler
 */
@TableField(value="user_id"，exist = false)
 private Long id;
```
2. @EnableEncryptableProperties
加密相关，默认自动开启
3. @ConfigurationPropertiesScan
配置文件注入的类不用写@Component
4. Swagger
5. Collections.emptyList(), CollectionUtils.containsAny，Collectors.groupingBy
返回不可变的空 List；判断两者是否有交集（可用stream代替）；
6. 全局异常拦截器  常见异常的输出日志信息方法 @Valid的MethodArgumentNotValidException异常
7. 什么时候加事务@Transactional
8. NPE预防x ：是包装类型 → 永远不要直接 !x、x && y；比较字符串 → 永远写 "xxx".equals(x)；判断空字符串 → 永远用 isEmpty / isBlank；判断集合空 → 永远用 CollectionUtils.isEmpty(list)；外部接口返回的对象字段 → 默认全部可能为 null
9.  线程池ThreadPoolTaskExecutor和@Async
10. 将前端传来的对象build另外一个对象，脱敏取关键值
11. Spring Security的passwordEncoder 
12. Caffeine  二级缓存 + Redis
13. LDAP 认证基础
14. 本地锁 vs 分布式锁
15. TransactionTemplate 
16. 分组校验 validator.validate()
17. Apache Commons Range 做区间判断

18. 
19. # lombok
# JPA
# 日期API
# Async 注解原理
# bean的循环依赖

# hutool工具类常用方法
# Digest spring自带加密方法类

# 数据安全
# AspectJ
# JWT问题实践
# SSO单点登录实践
# SSO跨域图解
# JDBC-Mybatis(TypeHandlers（类型处理器）,objectFactory,plugins)
# Optional实践
# Mybatis实战

# SpringSecurity,maven 2h，MybatisPus 1h+整理

# Mysql 21h
# linux 27h
# 设计模式 33h
# Redis 42h
# MQ,Kafka 12h
# Nginx 4h
# 微服务 44h
# Docker K8s  8h
# CI/CD
# 并发编程 32h
# JVM  68h
# 高可用高并发
# 分布式

# Easy Excel
# Java基础
## 1.Java 中的序列化和反序列化是什么？
1. 序列化（what）：将Java对象转成二进制字节流
2. where：程序执行时（类加载之后）
3. why：解决Java 对象只能存在 JVM 的内存里问题，使JVM关闭后仍可以在磁盘找到保存的对象，下次JVM可以从磁盘获取之前的对象 或者 网络传输 ，将数据从一个JVM 传到 另一个JVM
4. 使用场景：对象持久化，网络远程调用，缓存
5. how:实现Serializable接口，敏感字加transient关键字，显示定义serialVersionUID版本戳
```java
public class User implements Serializable {
    private static final long serialVersionUID = 1L;
    
    private String username;
    private transient String password; // 不参与序列化
    private int age;
}
```
6. 静态变量本身就不参与序列化。因为静态变量属于类级别，所有对象公用的变量，而序列化是将对象实例序列化
7. JDK原生序列化存在问题（性能差，不安全），一般用JSON，Protobuf，Hessian

## 2.Java 中 Exception 和 Error 有什么区别？
1. 常见异常和错误https://pic.code-nav.cn/mianshiya/question_picture/1814905001808924674/j0JcqCQh_image_mianshiya.webp   
2. 编译异常（checked Exception）可能出现的异常
3. 运行异常（unchecked Exception）编写错误导致的异常
4. 不会去捕获Error，因为JVM出现问题不可靠了，框架开发为了主线程不挂掉可能捕获Error
5. 不要在finally里面进行return，因为只能有一个return，会代替原来的return

## 3. 什么是 Java 的多态特性？
1. 编译看左边，运行看右边
2. 编译时多态（静态多态），方法重载
3. 运行时多态（动态多态），方法重写
4. 静态方法不能被重写
5. 变量的静态绑定，父类的访问不到子类的成员变量
```java
class Animal {
    public String name = "Animal";
}
class Dog extends Animal {
    public String name = "Dog";
}
public class Test {
    public static void main(String[] args) {
        Animal a = new Dog();
        System.out.println(a.name);  //输出Animal

    }
}
```

## 4.Java的优势
跨平台，生态丰富，gc，面向对象

## 5.Java 中的参数传递是按值还是按引用？
都是按值传递，传的都是副本

## 6. 为什么Java不支持多重继承
1. 避免菱形继承，就是如果子类继承的两个父类都有对他们共同父类方法的重写，子类不知道用哪个重写的方法
2. 对于接口多继承，有defualt默认的方法体的话，出现菱形继承，子类必须重写或者指定 父类.super.方法
3. 接口的defualt方法和抽象类的方法有什么区别 
   - defualt方法只有常量，抽象类能有变量
   - defualt多继承，抽象类单继承
   - defualt默认public修饰符，抽象类都能用    

## 7.Java 面向对象编程与面向过程编程的区别是什么？
1. POP 关注过程，OOP 关注对象；
2. POP 数据与函数分离，OOP 数据与行为封装在对象中；
3. OOP 有封装继承多态，更适合大型系统；POP 适合简单高性能任务。

## 8.Java 方法重载和方法重写之间的区别是什么？
1. 重载 返回类型可以不同 形参必须不同 可以重载静态方法 修饰符不受限制
2. 重写 返回类型必须相同或者返回父类的子类 形参必须相同 不能重写静态方法（静态属于类级） 修饰符不能比父类更严格

## 9.什么是 Java 内部类？它有什么作用？
1. 成员内部类
```java
public class OuterClass {
    private String outerField = "Outer Field";

    class InnerClass {
        void display() {
            //可以访问外部类所有成员
            System.out.println("Outer Field: " + outerField);
        }
    }
   
    public void createInner() {
        InnerClass inner = new InnerClass();
        inner.display();
    }
}
 //需要通过外部实例创建
OuterClass outer = new Outer();
OuterClass.InnerClass inner = outer.new InnerClass();
``
```
2. 静态内部类
```java
public class OuterClass {
    private static String staticOuterField = "Static Outer Field";
    //只能访问外部类的静态成员
    static class StaticInnerClass {
        void display() {
            System.out.println("Static Outer Field: " + staticOuterField);
        }
    }

    public static void createStaticInner() {
        StaticInnerClass staticInner = new StaticInnerClass();
        staticInner.display();
    }
}
//不需要通过外部实例创建，直接new
OutClass.StaticInnerClass inner = new OuterClass.StaticInnerClass()
```
3. 局部内部类
```java
class Outer {
    void test() {
        //定义在方法内的内部类
        class Inner {
            void show() {
                //能访问方法中的 final 或 effectively final 局部变量 或 外部类所有成员
                System.out.println("local inner");
            }
        }
        new Inner().show();
    }
}
```
4. 匿名内部类
```java
public class OuterClass {
    interface Greeting {
        void greet();
    }

    public void sayHello() {
        //没有类名，通常实现接口或继承抽象类
        Greeting greeting = new Greeting() {
            //能访问方法中的 final 或 effectively final 局部变量 或 外部类所有成员
            @Override
            public void greet() {
                System.out.println("Hello, World!");
            }
        };
        greeting.greet();
    }
}

```
```java
//Lambda表达式写匿名内部类
// 匿名内部类写法
Runnable r1 = new Runnable() {
    @Override
    public void run() {
        System.out.println("Hello");
    }
};

// Lambda 写法
Runnable r2 = () -> System.out.println("Hello");
```

## 10.Java8 有哪些新特性？
 java8.md
引入元空间代替永久代

## 11.Java11 有哪些新特性？
1. 标准化Http客户端API
2. String方法增强
```java
//1.isBlank和isEmpty区别
//isEmpty() —— 判断字符串长度是否为 0
"".isEmpty();        // true
"  ".isEmpty();      // false（因为长度是 2）
//isBlank() —— 判断是否只包含空白
"".isBlank();        // true
"  ".isBlank();      // true（只有空格，也算 blank）
"\t\n".isBlank();    // true（tab、换行 都算 blank）

//2.strip()和trim()区别
//strip()去掉所有 Unicode 空白 而 trim()只去 ASCII 空白

//3.lines() 分割参照：字符串中的“行终止符”：\n、\r、\r\n
// 将字符串按行分割成 Stream
multiLine.lines()
    .map(line -> "处理: " + line)
    .forEach(System.out::println);

long lineCount = multiLine.lines().count();
System.out.println("总行数: " + lineCount);

//repeat()
System.out.println("Java ".repeat(3)); // "Java Java Java "
System.out.println("=".repeat(50));    // 50个等号
System.out.println("*".repeat(0));     // 空字符串
```
3. File文件读写简化
```java
// 写入文件
String content = "这是一个测试文件\n包含多行内容\n中文支持测试";
//参数：路径，内容，编码 返回值：路径
Path tempFile = Files.writeString(
    Paths.get("temp.txt"), 
    content,
    StandardCharsets.UTF_8
);

// 读取文件
//参数：路径，编码  返回值：内容
String readContent = Files.readString(tempFile, StandardCharsets.UTF_8);
System.out.println("读取的内容:\n" + readContent);
```
```java
//读取大文件：流式处理
try (Stream<String> lines = Files.lines(tempFile)) {
    lines.filter(line -> !line.isBlank())
         .map(String::trim)
         .forEach(System.out::println);
}

```
4. Optional新增isEmpty
## 12.Java17 有哪些新特性？
1. Sealed密封类:控制类的继承
```java
//父类
public sealed class Shape 
    // 只允许这三个类继承
    permits Circle, Rectangle, Triangle {
}
//子类继承后要选择一种继承策略
//final:该子类不能再被继承了
public final class Circle extends Shape {
}

//sealed：和父类一样选择谁能继承我
public sealed class Triangle extends Shape 
    permits RightTriangle {
}

//non-sealed：谁都能继承我
public non-sealed class Rectangle extends Shape {
}
```
2. 增强的伪随机数生成器
3. 强封装JDK内部API:彻底禁止通过反射访问JDK内部类
## 13.Java21 有哪些新特性？
1. Virtual Threads虚拟线程
只要发生线程等待，该线程就可以去执行其他的任务，适用于高并发，IO密集型任务
2. Switch模式匹配
```java
//原来代码
public String processMessage(Object message) {
    if (message instanceof String) {
        String textMessage = (String) message;
        return "文本消息：" + textMessage;
    } else if (message instanceof Integer) {
        Integer numberMessage = (Integer) message;
        return "数字消息：" + numberMessage;
    } else if (message instanceof List) {
        List<?> listMessage = (List<?>) message;
        return "列表消息，包含 " + listMessage.size() + " 个元素";
    } else {
        return "未知消息类型";
    }
}
//Switch模式匹配：匹配类型，若是赋值给形参执行逻辑
public String processMessage(Object message) {
    return switch (message) {
        case String text -> "文本消息：" + text;
        case Integer number -> "数字消息：" + number;
        case List<?> list -> "列表消息，包含 " + list.size() + " 个元素";
        case null -> "空消息";
        default -> "未知消息类型";
    };
}
// 增加判断条件：根据字符串长度采用不同处理策略
public String processText(String text) {
    return switch (text) {
        case String s when s.length() < 10 -> "短文本：" + s;
        case String s when s.length() < 100 -> "中等文本：" + s.substring(0, 5);
        case String s -> "长文本：" + s.substring(0, 10);
    };
}

```
3. Record模式
```java
//定义record
public record Person(String name, int age) {}
public record Address(String city, String street) {}
public record Employee(Person person, Address address, double salary) {}
//类型判断同时进行赋值解构
public String analyzeEmployee(Employee emp) {
    return switch (emp) {
        // 一次性提取所有需要的信息
        case Employee(Person(var name, var age), Address(var city, var street), var salary) 
            when salary > 50000 -> 
            String.format("%s（%d岁）是高薪员工，住在%s%s，月薪%.0f", 
                         name, age, city, street, salary);
        case Employee(Person(var name, var age), var address, var salary) -> 
            String.format("%s（%d岁）月薪%.0f，住在%s", 
                         name, age, salary, address.city());
    };
}
```
4. 有序集合
```java
//补充几个集合方法
List<String> tasks = new ArrayList<>();
tasks.addFirst("鱼皮的任务");    // 添加到开头
tasks.addLast("小阿巴的任务");   // 添加到结尾

String firstStr = tasks.getFirst();  // 获取第一个
String lastStr = tasks.getLast();   // 获取最后一个

String removedFirst = tasks.removeFirst();  // 删除并返回第一个
String removedLast = tasks.removeLast();    // 删除并返回最后一个

List<String> reversed = tasks.reversed();   // 反转列表
```
5. 分代ZGC
分年轻代和老年代（短命区和长寿区）

## 14.Java 中 String、StringBuffer 和 StringBuilder 的区别是什么？
1. 字符串基本不变，只有少量拼接，用String
2. 多线程环境频繁修改字符串，用StringBuffer    线程安全 synchronized加在了整个方法上
3. 单线程下大量拼接操作，用StringBuilder      
```java
//java编译器会将简单的字符串拼接优化成StringBuilder  但是在循环中则会反复创建新的StringBuilder  
String s = "";
for (int i = 0; i < 1000; i++) {
    s += i;   // 为什么不推荐？
}
//正确做法
StringBuilder sb = new StringBuilder();
for (int i = 0; i < 1000; i++) {
    sb.append(i);
}
String s = sb.toString();
```
4. 其他：JDK9之后这三个底层的char数字全换成了byte数组，同时加了个coder字段标记编码方式

### 为什么String不可变
1. 字符串常量池能生效，JVM专门开了一篇区域存字符串常量，一旦可变，别的引用指向的内容一同会变
2. 哈希值可以缓存
3. 天然线程安全

## 15.Java 的 StringBuilder 是怎么实现的？
```java
abstract class AbstractStringBuilder {
    char[] value;  // 可扩容字符数组
    int count;     // 实际存了多少个字符
}
```
### 添加方法append流程
1. 计算要添加的长度
2. 判断容量够不够
3. 不够进行扩容  乘2加2 降低频繁触发扩容数组拷贝
4. 够将内容拷贝到数组中
5. 更新count

# JVM基础
## 编译，编译型与解释型语言
1. 编译是将人写的源码编译成程序看得懂的语言
2. java的编译是通过编译器编译成字节码让JVM解释直接执行   <mark> 编译器+jvm解释器（JIT）<mark>
3. C/C++的编译是编译成机器码直接让cpu解释执行   <mark>编辑器<mark>
4. python是通过解释器直接边读边执行   <mark>解释器<mark>
5. 编译型语言相对解释型语言 启动慢但运行快 不能跨平台（机器码依赖于平台本身） 调试起来相对麻烦（要重新编译）

## 组织架构
### 类加载器
1. 将字节码文件加载到内存当中
### 运行时数据区
1. 存放数据
### 执行引擎
1. 解释器，JIT,GC

## 类加载机制
整个生命周期分为七个阶段，分别是加载、验证、准备、解析、初始化、使用和卸载。其中验证、准备和解析这三个阶段统称为连接。除去使用和卸载，就是 Java 的类加载过程。
### Loading（载入）
将字节码从数据源变成二进制字节流存入内存
### Verification（验证）
对二进制字节流进行校验
### Preparation（准备）
为静态变量（static 关键字修饰的）分配空间，赋默认值
### Resolution（解析）
1. 将常量池中的符号引用转化为直接引用。
2. 符号引用：用符号代表引用关系
3. 直接引用：直接引用实际内存地址
4. 如果有动态加载，解析会推迟到初始化之后。因为符号转为直接引用的内存地址不是静态固定的
### Initialization（初始化）
执行类构造器方法

## 类加载器
1. 引导类加载器（Bootstrap ClassLoader）：负责加载 JVM 基础核心类库，如 rt.jar、sun.boot.class.path 路径下的类。
2. 扩展类加载器（Extension ClassLoader）：负责加载 Java 扩展库中的类，例如 jre/lib/ext 目录下的类或由系统属性 java.ext.dirs 指定位置的类。
3. 系统（应用）类加载器（System ClassLoader）：负责加载系统类路径 java.class.path 上指定的类库，通常是你的应用类和第三方库。
4. 用户自定义类加载器：Java 允许用户创建自己的类加载器，通过继承 java.lang.ClassLoader 类的方式实现。这在需要动态加载资源、实现模块化框架或者特殊的类加载策略时非常有用。
### 双亲委派模型
类加载器永远会先让父类加载器进行加载。  <br/>
1. 安全：不会自己写个String类库让底部类加载器执行，以后就用自己写的String类库。因为永远是Bootstrap ClassLoader去加载核心类库，而且会去特定安全区域找，不会找到自己写的String类库
2. 类不会被重复加载（父类加载过的子不会再加载）
