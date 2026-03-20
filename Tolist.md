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
2. 重写 返回类型必须相同或者返回父类的子类 形参必须相同 不能重写静态方法（静态属于类级） 修饰符不能超过父类范围
3. 

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
