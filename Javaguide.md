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
singleton : IoC 容器中只有唯一的 bean 实例。Spring 中的 bean 默认都是单例的，是对单例设计模式的应用。项目启动时就创建。
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

# 什么时候省略@Autowired
```java
//1.一个构造器时，里面的形参不用@Autowired
@Component
public class UserService {
    // 不需要@Autowired
    public UserService(UserRepository repository, RoleService roleService) {
        this.repository = repository;
        this.roleService = roleService;
    }
}

//2.@Configuration 类中的Bean方法调用
@Configuration
public class AppConfig {
    
    @Bean
    public DataSource dataSource() {
        return new DataSource();
    }
    
    @Bean
    // 不需要@Autowired，直接调用方法
    public JdbcTemplate jdbcTemplate() {
        return new JdbcTemplate(dataSource());
    }
}

//3.测试类中的字段注入
@SpringBootTest
class UserServiceTest {
    // 不需要@Autowired（Spring Boot 2.1+）
    private UserService userService;
    
    // 或者构造器注入
    public UserServiceTest(UserService userService) {
        this.userService = userService;
    }
}

//4.加上@bean方法的形参
@ServletComponentScan
@EnableScheduling
@SpringBootApplication
public class TliasWebManagementApplication {

    public static void main(String[] args) {
        SpringApplication.run(TliasWebManagementApplication.class, args);
    }

    @Bean
    //AliyunOSSProperties不用@Autowired
    public AliyunOSSOperator aliyunOSSOperator(AliyunOSSProperties ossProperties) {
        return new AliyunOSSOperator(ossProperties);
    }
}
```
# MVC
- DispatcherServlet：核心的中央处理器，负责接收请求、分发，并给予客户端响应。
- HandlerMapping：处理器映射器，根据 URL 去匹配查找能处理的 Handler ，并会将请求涉及到的拦截器和 Handler 一起封装。
- HandlerAdapter：处理器适配器，根据 HandlerMapping 找到的 Handler ，适配执行对应的 Handler；
- Handler：请求处理器，处理实际请求的处理器。
- ViewResolver：视图解析器，根据 Handler 返回的逻辑视图 / 视图，解析并渲染真正的视图，并传递给 DispatcherServlet 响应客户端

流程说明（重要）：<br>
1. 客户端（浏览器）发送请求， DispatcherServlet拦截请求。DispatcherServlet 根据请求信息调用 HandlerMapping 。<br>
2. HandlerMapping 根据 URL 去匹配查找能处理的 Handler（也就是我们平常说的 Controller 控制器） ，并会将请求涉及到的拦截器和 Handler 一起封装。<br>
3. DispatcherServlet 调用 HandlerAdapter适配器执行 Handler 。<br>
4. Handler 完成对用户请求的处理后，会返回一个 ModelAndView 对象给DispatcherServlet。前后端分离到这返回一个JSON<br>
5. ModelAndView 顾名思义，包含了数据模型以及相应的视图的信息。Model 是返回的数据对象，View 是个逻辑上的 View。ViewResolver 会根据逻辑 View 查找实际的 View。DispaterServlet 把返回的 Model 传给 View（视图渲染）。把 View 返回给请求者（浏览器）

## 16.MySQL 中的日志类型有哪些？binlog、redo log 和 undo log 的作用和区别是什么？
redo log:记录数据在页的修改，保证数据不丢失，循环写（写满从头覆盖）
undo log:记录旧值，用于回滚，随事物变化形成版本链
binlog:记录sql操作，用于主从同步，Server层日志，追加写

顺序：先改内存数据，然后写redo日志，再去修改磁盘

## 17.MySQL事务是怎么实现的
事务 AIDC
redo log:保证持久性
undo log:保证原子性
锁：保证隔离性
MCVV：保证隔离性的读写并发
一致性：由其他性共同实现
### redo log （Duration）
#### 参考：https://juejin.cn/post/7213296062085644344
1. WAL策略（Write ahead logging）：先写日志再写数据
2. 两个指针，write pos指向日志写到哪了，checkpoint指向已经刷盘的位置，两指针中间是待刷盘的脏数据
3. 流程
SQL
 ↓
undo log（回滚用）
 ↓
数据页改内存    InnoDB 内存里改数据页
 ↓
redo log 生成
 ↓
redo buffer(InnoDB 内存)
 ↓
commit 时：
   → write redo log
   → （由 innodb_flush_log_at_trx_commit 控制）
 ↓
返回成功 
 ↓
后台慢慢刷数据页
4. innodb_flush_log_at_trx_commit   
write: Redo Log Buffer → OS Page Cache
fsync(刷盘)：把 OS Page Cache 里的日志强制同步到物理磁盘

0：提交事务时，不做 write + fsync；
后台线程大约每秒做一次日志写出并同步（因此最多可能丢失约 1 秒事务）
优点：性能最高。
风险：实例崩溃时可能丢失最多1秒的事务数据。

1（默认值）：每次提交都 write + fsync
优点：完全遵守ACID特性，数据安全性最高。
缺点：性能开销较大，尤其在高并发场景下。

2：每次提交只 write 到 OS Cache，每秒 fsync
优点：性能优于1，数据安全性高于0。
风险：实例崩溃时可能丢失最近1秒内的事务数据。

0:延迟写。效率最高,最不安全:log_buff  —> 每隔1秒   —>  os cache  —>实时---—>  disk
1:实时写,实时刷。效率最低,最安全:log_buff  —>   实时    —>  os cache  —>实时---—>  disk
2:实时写,延迟刷。效率折中,安全折中:log_buff  —>   实时    —>  os cache  —>每隔1秒—>  disk

### undo log
每条数据有两个隐藏字段
trx_id 指向最后修改这条记录的事务的id
roll pointer 指向上一个版本

```txt
初始张三，事务100改成李四，事务200改成王五
当前数据页：name='王五', trx_id=200, roll_pointer →
    undo log：name='李四', trx_id=100, roll_pointer →
        undo log：name='张三', trx_id=0, roll_pointer=null
```

## 18.长事务会导致什么问题
锁竞争激烈，容易死锁，undo log膨胀，主从延迟

### 删除大量数据
1. 根据问题查出记录的索引，走索引删
2. 将要的数据插入到新表

## 19.MVCC
多版本并发控制，无锁读机制
###  脏读 / 不可重复读 / 幻读
脏读：一个事务读到了另一个事务“尚未提交”的数据
不可重复读：同一个事务内，对同一行数据两次查询，结果不一致 （能读到其他事务对 同行数据 的修改）
幻读：同一个事务内，用同样的条件查询，两次结果集“行数变了” （能读到其他事务对 行数 的修改）
### 快照读和当前读
快照读：不加锁的select，走MVCC，通过 Read View 从版本链中选择可见版本。
当前读：加锁的select和增删改，直接读取最新已提交数据，不走MVCC
### 隔离级别
图：https://www.mianshiya.com/bank/1791003439968264194/question/1780933295484203009#heading-9
### Read View（
用于判断当前事务读取的版本链（undo log）是否可见

Read View = {
    creator_trx_id    // 当前事务ID
    m_ids             // 正在执行，还没提交的事务集合
    min_trx_id        // m_ids 中最小的
    max_trx_id        // 系统即将分配的下一个事务ID
}
#### 可见性判断规则，事务对应版本
1. trx_id == creator_trx_id   undo log的版本id和当前事务一致，可见
2. trx_id < min_trx_id        执行完毕的事务，可见
3. trx_id ∈ m_ids             活跃事务，不可见
4. trx_id >= max_trx_id       未来事务，不可见

### RC与RR
RC 每次 SELECT 都创建新的 Read View，所以可能读到新数据 → 不可重复读
RR 只在事务中第一次执行的“普通 SELECT（快照读）创建 Read View，所以一直读同一个版本 → 可重复读 


### RR与幻读 todo 修改并指出错误，分清楚是几个事务，列出可能的情况
RR理论不能完全避免幻读，但innodb默认有间隙锁
```sql
--事务开始
BEGIN

--语句一
SELECT * FROM orders WHERE amount > 100
--语句二
SELECT * FROM orders WHERE amount > 100 for update

--语句三
-- 当前读，所加在外部的事务上
UPDATE orders SET amount=150 WHERE id=2;


--语句四
SELECT * FROM orders WHERE amount > 100
--语句五
SELECT * FROM orders WHERE amount > 100 for update
COMMIT

```
1. 一三四，RR情况下，语句一和语句三是可以并发执行的，因为语句一是快照读，语句三是当前读。然后语句四继续读的是快照版本，所以不会产生幻读
2. 一三五，语句五变成了当前读，读最新的数据，会发生类似幻读，五是单独事务
3. 二三四，语句二会加间隙锁，导致语句三阻塞无法修改，语句四再去查还是原来的数据，不会产生幻读；就算语句四在语句三之后执行，如果之前有快照版本，语句四查的是快照版本，语句三不会产生影响，如果之前没有快照版本，语句四能查到语句三的修改，但这是新的快照，不能称之为幻读。
4. 二三五，二和五可以一起执行，不会出现幻读；就算五再三后执行，因为不是同一个事务，只能是类似幻读。


## 20.锁
根据粒度分：表锁或行锁，粒度越细并发越高
根据模式分：共享锁（S锁）或排他锁（X锁）
### 行锁
1. 记录锁：只锁一行    锁id=5
2. 间隙锁：锁两条记录之间的间隙，解决幻读   锁id (5,8)
3. 临键锁（Next-Key Lock）：记录锁 + 间隙锁  锁id【5,8）
### 共享锁（S锁）和排他锁（X锁）
1. 共享锁：可以多个事务读，不能写
2. 排他锁：只能自己读写
### 元数据锁（表结构锁）
DDL（Data Definition Language）:操作“表结构”
DML（Data Manipulation Language）: 操作“表数据”

当执行 DDL 时，需要获取 MDL 写锁（独占锁），会阻塞所有其他事务对该表的读写操作，直到表结构修改完成。
当执行 DML 时，会获取 MDL 读锁（共享锁），允许多个事务同时访问数据，但不能执行 DDL；
### 意向锁（只作为一个标记）
1. 为什么元数据锁（表锁）和行锁会冲突？
   因为行锁锁一行，表锁锁整张表，表锁锁整张表的时候不允许表中已有部分数据被其他锁占用

IS共享意向锁：当表中需要行锁S锁时，在表上加个IS锁，表示表中有行锁
IX独占意向锁：当表中需要行锁X锁时，在表上加个IX锁，表示表中有行锁

当这个表要上其他表级别的S锁时，可查看是否有IX锁，没有就可以上表级别的S锁
当这个表要上其他表级别的X锁时，可查看是否有IX锁或IS锁，没有就可以上表级别的X锁

2. 为什么要有意向锁
   作为标记，用于快速判断表中是否有行锁，而不用遍历表中所有数据

3. 为什么IS,IX不会冲突
   IS，IX只作为标记。并且表中可以存在对不同行的S锁和X锁

### 行锁
1. 锁的是索引（记录锁）                           WHERE id = 1   （唯一索引 + 精确匹配）
2. 会变成范围锁next-key lock（临键锁）            WHERE name = 'A'（非唯一索引）
3. 会锁全表                                      WHERE name = 'A'（非索引）

```txt
假设索引是：
id: 1   5   10   20

1.SELECT * FROM t WHERE id = 10 FOR UPDATE;   id是非唯一索引
锁(5, 10]   ← 前开后闭

2.SELECT * FROM t WHERE id > 10 FOR UPDATE;
锁(10, +∞)

3.SELECT * FROM t WHERE id >= 10 AND id < 20 FOR UPDATE;
锁(5,20)
```
### 插入意向锁
间隙锁（gap lock）:锁区间，防止插入
Insert Intention Lock：多个事务可能想在同一个 gap 插入不同的数据（前提是不插同一个位置）

问题：间隙锁的间隙是怎么确定的？
根据索引排序确定

### AUTO-INC（自增锁）表级锁
AUTO-INC lock 在逻辑上是表级的序列控制机制，但在实现上是通过轻量级互斥量（mutex）保护自增计数器的更新操作，是 InnoDB 为了保证 AUTO_INCREMENT 主键生成唯一、正确而加的一种“表级序列控制锁”

#### innodb_autoinc_lock_mode
模式0：拿 AUTO-INC 锁 → 执行完整 INSERT → 释放锁   auto-inc
模式1：普通单条INSERT快速拿id释放锁（互斥量）；批量INSERT知道行数一次申请一段ID，互斥量一次插入全部再释放锁；批量INSERT不知道行数，auto-inc保证id连续自增
模式2：并发执行，可能会因回滚导致跳号   互斥量

不知道插入行数的sql
```sql
INSERT INTO t (col1, col2)
SELECT col1, col2 FROM other_table;

INSERT INTO t
SELECT a.col1, b.col2
FROM a JOIN b ON a.id = b.id;

INSERT INTO t
SELECT a.col1, b.col2
FROM a JOIN b ON a.id = b.id;

INSERT INTO t
SELECT * FROM (
  SELECT * FROM a LIMIT 1000 -- 行数最多1000行
) AS tmp;
```
### 悲观锁和乐观锁
 悲观锁：先假设一定会冲突，先加锁再操作
 ```sql
-- 悲观锁示例：扣减库存
BEGIN;
-- FOR UPDATE先锁住这行，别人想改就得等
SELECT stock FROM products WHERE id = 1 FOR UPDATE;
-- 业务判断库存够不够
-- 扣减库存
UPDATE products SET stock = stock - 1 WHERE id = 1;
COMMIT;
 ```
 乐观锁：假设大概率不冲突，先操作，冲突了再处理
```sql
-- 1.读取数据和版本号
SELECT id, name, stock, version FROM products WHERE id = 1;
-- 假设读出来 version = 5

-- 更新时带上版本号
UPDATE products
SET stock = stock - 1, version = version + 1
WHERE id = 1 AND version = 5;

-- 检查影响行数
-- 如果返回 0 说明被别人改过，需要重试或报错

-- 2.假设读出来 stock = 100,存在ABA问题
UPDATE products
SET stock = 99
WHERE id = 1 AND stock = 100;
```
### 死锁
满足条件：互斥条件，请求并持有，不可剥夺，循环等待
1. innodb会自动检测死锁，回滚代价最小事务；
2. 设置锁等待超时，超时后锁自动放弃并回滚 SET innodb_lock_wait_timeout = 5;
3. 手动查看信息，杀死事务 SHOW ENGINE INNODB STATUS;
4. 拆分事务，按顺序

## 21.MySQL 中 count(*)、count(1) 和 count(字段名) 有什么区别？
1. 统计包含null的行数  count(1) count(*)
2. 统计不包含null的行数 count(字段名)
3. count(*)找行数走的是最小的索引树，索引树的叶子节点存的页，叶子节点里的记录数是行数，但要注意不同事务看到的“可见行数”不同

### 大表count优化
COUNT(*) = 扫索引 + 判断可见性 + 累加
可用缓存维护计数

## 22.sql优化
1. 命中索引
2. 减少回表
3. 减少I/O

## 23.如何在 MySQL 中避免单点故障?
1. 单点故障指的是只有一台实例，一旦宕机会导致整个系统不可用
2. 冗余（多实例），复制（数据同步），故障切换（自动恢复）
3. 主从复制，读写分离（主库写，从库读）
4. MHA 和 MGR

### 主从复制
流程：
1. 主库dump线程监听binlog变更，有新内容就推给从库
2. 从库的I/O线程拉取主库数据，将binlog写进本地的relay log
3. 从库sql线程读取relay log，执行同步sql

同步模式：
1. 异步复制（默认）
2. 同步复制
3. 半同步复制：至少一个从库确认

binlog的三种格式
1. statement:记录sql语句本身，日志小但可能会因为函数执行结果不同导致主从不一致，比如now()
2. row:记录日志前后变化，日志大但绝对一致，生产推荐row
3. mixed:mysql自行判断，一般用statement，遇到不安全的语句用row

### 读写分离
1. 代码层封装 AbstractRoutingDataSource配合AOP，问题是主从切换需要改配置重启，每个语言都要一套
2. 中间层代理 ShardingSphere-Proxy（还支持分库分表） 或 ProxySQL（性能最好）, 问题是多一层要维护 
ps:如果事务里有读有写，一般同一走主库

### 主从延迟
1. 对一致性要求高的走主库
2. 短时间的写后读走主库，延迟窗口过了走从库
3. 并行复制，开多个线程让从库尽快同步主库
4. 写完主库写缓存（缓存一致性问题）

其他：
1. 监控延迟，SHOW SLAVE STATUS里的Seconds_Behind_Master不准确，网络中断了也是0。用pt-heartbeat工具。
2. 级联复制：从库挂从库，延迟会增加

### 分库分表
1. 水平分表：表结构字段一样，数据不同
2. 垂直分表：常用的字段和不常用字段垂直拆分成不同的表
3. 水平分库：表结构分到多个数据库，数据按规则分散存储
4. 垂直分库：按照不同的业务分成不同的数据库

解决问题：
1. 分表解决单条数据量太大导致查询慢的问题
2. 分库解决单机资源瓶颈，将压力分摊到多台服务器

分片键
1. 垂直：业务（分库），常用字段（分表）
2. 水平：哈希取模（比如id模4，加库重新大量数据），范围分片（比如按时间，最新时间的库压力大），一致性哈希

代价：
1. 跨库join：原本一条sql,现在要查多次在应用层组装   
2. 分布式事务
3. 全局唯一ID：分布式ID，如Snowflake,Leaf
4. 跨分片聚合：count,order by得从不同分片拉取合并
5. 要运维的库多

解决：
1. 跨库join     宽表同步到ES，数据冗余，es一致性
2. 事务一致性    2PC两阶段提交;TCC模式；本地消息表
3. 排序分页      中间键


中间件
ShardingSphere-JDBC(小公司，有代码入侵),ShardingSphere-Proxy（大公司）,MyCat

实施流程：
1. 评估
2. 设计分片：一次查询的条件不能涉及全部分片；多次查询不能集中在一个分片
3. 选择中间件
4. 数据迁移（Canal监听binlog）
5. 灰度切换，先让新库去读，再试写。期间保持双写。

24. 从 MySQL 获取数据，是从磁盘读取的吗？（buffer pool）




数据结构，redo log流程图，RR与幻读，MySQL 二级索引有 MVCC 快照吗？，	如果 MySQL 中没有 MVCC，会有什么影响？，	MySQL 默认的事务隔离级别是什么？为什么选择这个级别？，explain
业务流程：
电子签
## 临时设置环境变量
$env:JIB_USERNAME="clu57"
$env:JIB_PASSWORD="x"
