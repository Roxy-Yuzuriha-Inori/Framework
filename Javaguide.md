https://javaguide.cn/system-design/framework/spring/spring-knowledge-and-questions-summary.html#bean-%E6%98%AF%E7%BA%BF%E7%A8%8B%E5%AE%89%E5%85%A8%E7%9A%84%E5%90%97
# 什么是IOC
控制反转，将创建对象的权力交给spring容器。<br>
好处：降低耦合，更容易管理资源<br>
场景：Service层要调用Dao层，现有Service层的impl和接口，Dao的impl和接口<br>
不用IOC情况下，直接new Dao层的impl（不能new接口）<br>
接口名 变量名 = new 实例名（）<br>
导致问题：实例名实例对象一旦改变，Service层所有new的步骤全部都得改变

用IOC情况下，DI注入
接口名 变量名 

# 将一个类声明为 Bean 的注解有哪些?
- @Repository : DAO层<br>
- @Service : Service层<br>
- @Controller : controller层<br>
- @Component：随便什么层,作用于类，该类作为一个bean,类中方法返回的对象不会作为bean<br>
 - @Component通常是通过类路径扫描来自动侦测以及自动装配到 Spring 容器中（我们可以使用@ComponentScan 注解定义要扫描的路径从中找出标识了需要装配的类自动装配到 Spring 的 bean 容器中）。<br>
 - @Bean:作用于方法，方法返回的第一个对象作为bean

# 注入 Bean 的注解有哪些？
## @Autowired 
``` java
// SmsService 接口有两个实现类：SmsServiceImpl1、SmsServiceImpl2（均被 Spring 管理）

// 报错：byType 匹配到多个 Bean，且属性名 "smsService" 与两个实现类的默认名称（smsServiceImpl1、smsServiceImpl2）都不匹配
@Autowired
private SmsService smsService;

// 正确：属性名 "smsServiceImpl1" 与实现类 SmsServiceImpl1 的默认名称匹配
@Autowired
private SmsService smsServiceImpl1;

// 正确：通过 @Qualifier 显式指定 Bean 名称 "smsServiceImpl1"
@Autowired
@Qualifier(value = "smsServiceImpl1")
private SmsService smsService;

```
@Autowired 是 Spring 提供的注解，@Resource 是 JDK 提供的注解。Autowired 默认的注入方式为byType（根据类型进行匹配），@Resource默认注入方式为 byName（根据名称进行匹配）。当一个接口存在多个实现类的情况下，@Autowired 和@Resource都需要通过名称才能正确匹配到对应的 Bean。Autowired 可以通过 @Qualifier 注解来显式指定名称，@Resource可以通过 name 属性来显式指定名称。@Autowired 支持在构造函数、方法、字段和参数上使用。@Resource 主要用于字段和方法上的注入，不支持在构造函数或参数上使用。

## @Resource
``` java
// 报错，byName 和 byType 都无法匹配到 bean
@Resource
private SmsService smsService;
// 正确注入 SmsServiceImpl1 对象对应的 bean
@Resource
private SmsService smsServiceImpl1;
// 正确注入 SmsServiceImpl1 对象对应的 bean（比较推荐这种方式）
@Resource(name = "smsServiceImpl1")
private SmsService smsService;

```
# 注入 Bean 的方式有哪些？
1.构造函数注入：通过类的构造函数来注入依赖项。
> 如果一个类有多个构造器，则需要@Autowired指定一个构造器用来表明，这个类的bean用这个构造器创建;
> 如果一个类只有一个构造器，则无需添加@Autowired
2.Setter 注入：通过类的 Setter 方法来注入依赖项。@Resource
3.Field（字段） 注入：直接在类的字段上使用注解。@Resource

## 构造器注入（推荐）
正常来讲，要创建这个对象应该要new一个userRepository给这个构造器，但是这个类因为@Service由spring管理，这个bean无需new也无需new userRepository
```java
@Service
public class UserService {

    private final UserRepository userRepository;

    public UserService(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}

//这里有两个bean,并且前提是UserRepository已经成为了bean
```
## setter注入
```java
@Service
public class UserService {

    private UserRepository userRepository;

    // 在 Spring 4.3 及以后的版本，特定情况下 @Autowired 可以省略不写
    @Autowired
    public void setUserRepository(UserRepository userRepository) {
        this.userRepository = userRepository;
    }

    //...
}
```
## Field（字段） 注入
```java
@Service
public class UserService {

    @Autowired
    private UserRepository userRepository;

}
```
# Bean 的作用域有哪些?
singleton : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。
prototype : 每次获取都会创建一个新的 bean 实例。也就是说，连续 getBean() 两次，得到的是不同的 Bean 实例。
request （仅 Web 应用可用）: 每一次 HTTP 请求都会产生一个新的 bean（请求 bean），该 bean 仅在当前 HTTP request 内有效。
session （仅 Web 应用可用） : 每一次来自新 session 的 HTTP 请求都会产生一个新的 bean（会话 bean），该 bean 仅在当前 HTTP session 内有效。
application/global-session （仅 Web 应用可用）：每个 Web 应用在启动时创建一个 Bean（应用 Bean），该 bean 仅在当前应用启动时间内有效。
websocket （仅 Web 应用可用）：每一次 WebSocket 会话产生一个新的 bean
## 配置bean的作用域
``` xml
<bean id="..." class="..." scope="singleton"></bean>
```
``` java
@Bean
@Scope(value = ConfigurableBeanFactory.SCOPE_PROTOTYPE)  //多例bean
public Person personPrototype() {
    return new Person();
}
```
# Bean 是线程安全的吗？
singleton 单例可能存在（有状态的bean存在安全问题，无状态的不存在安全问题）
prototype 多例每次获取都会创建一个新的 bean 实例，不存在资源竞争问题，不存在线程安全问题。

## 有状态的bean:存在可变字段
``` java
// 定义了一个购物车类，其中包含一个保存用户的购物车里商品的 List
@Component
public class ShoppingCart {
    private List<String> items = new ArrayList<>();
//item是一个可变字段
    public void addItem(String item) {
        items.add(item);
    }

    public List<String> getItems() {
        return items;
    }
}
```
## 解决单例bean的线程安全问题
1.避免可变成员变量: 尽量设计 Bean 为无状态。
2.使用ThreadLocal: 将可变成员变量保存在 ThreadLocal 中，确保线程独立。
3.使用同步机制: 利用 synchronized 或 ReentrantLock 来进行同步控制，确保线程安全。

