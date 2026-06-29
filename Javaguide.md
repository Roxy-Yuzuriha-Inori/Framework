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

## 24. 从 MySQL 获取数据，是从磁盘读取的吗？
1. 从buffer pool读
2. 数据是以“页”为单位读取，buffer pool最小存储单位是页
3. 淘汰算法LRU（最近最少使用）

## 25.MySQL 的 Doublewrite Buffer 是什么？它有什么作用？
1. Doublewrite Buffer 是 InnoDB 为了解决“部分页写入（Partial Page Write）”问题而设计的写入保护机制
2. 部分页写入（Partial Page Write）：将16kb数据库的页写到磁盘的时候中断了，导致数据库的页损坏了
3. 数据库的页损坏：因为写了一个页（16kb）一半数据，里面一半标记为新数据一半旧数据，页结构被破坏。
4. 解决：先用Doublewrite Buffer 顺序写 一份新的，坏了的从Doublewrite Buffer 拷贝
5. 为什么不增量修复（在读损坏页的时候忽略已写入的半页）？页是最小单位。每个边修边写成本大
6. 对比redolog,redolog解决的是数据丢失，Doublewrite解决的是页损坏
redoLog
写数据
↓
redo log ✅
↓
提交成功 ✅（用户看到成功）
↓
数据页还在内存 ❗（没写磁盘）

此时宕机 ❌

## 26.MySQL 中的 Log Buffer 是什么？它有什么作用？
1. 用于缓存redoLog，将每次事务的redoLog缓存在此，之后一起刷盘
2. 刷盘时机：事务提交；logBuffer使用量满；innodb后台线程定时刷盘

## 27.为什么在 MySQL 中不推荐使用多表 JOIN？
1. MySQL执行JOIN的本质是 拿驱动表的每一行（全表扫描）去 被驱动表查询匹配（可走索引）
2. 小表做驱动表  大表做被驱动表
3. 代替方案：应用层做JOIN，缓存等

## 28.MySQL 中如何解决深度分页的问题？
1. 子查询优化
```sql
-- 原始写法，慢  查询了前999条所有数据
SELECT * FROM mianshiya
WHERE name = 'yupi'
ORDER BY id
LIMIT 99999990, 10;

-- 优化写法   分两次查询，第二次查询id+name作为索引，拿到id再用主键索引取后十条完整记录
SELECT * FROM mianshiya
WHERE name = 'yupi'
  AND id >= (
    SELECT id FROM mianshiya
    WHERE name = 'yupi'
    ORDER BY id
    LIMIT 99999990, 1
  )
ORDER BY id
LIMIT 10;

```
2. 游标分页
MySQL可以通过索引跳到起始位置，不要扫描前面的数据。但这样只能连续翻页。
```sql
-- 第一页
SELECT * FROM mianshiya WHERE name = 'yupi' ORDER BY id LIMIT 10;
-- 假设第一页的最后一条 id 是 100

-- 第二页  第二次查询带上上次的id作为起点
SELECT * FROM mianshiya WHERE name = 'yupi' AND id > 100 ORDER BY id LIMIT 10;

```
3. ES深度分页

## 29.如何在 MySQL 中监控和优化慢 SQL？
监控：
```sql
--四个参数
slow_query_log = 1                    -- 启用慢查询日志
slow_query_log_file = /var/log/mysql/mysql-slow.log  -- 日志文件路径
long_query_time = 2.0                 -- 超过 2 秒记录
log_queries_not_using_indexes = 1     -- 没走索引的也记录
```
优化：explain语句分析单条sql,skywalking等线上工具

## 30.MySQL 中 DELETE、DROP 和 TRUNCATE 的区别是什么？
1. DELETE:删数据，保留表结构，可回滚，DML
2. DROP：删数据，删表结构，不可回滚,DDL
3. TRUNCATE：删数据，保留表结构，不可回滚,DDL

### DELETE
1. 逐行删除数据，实际打删除标记，不会立马删除
2. 记录redoLog,undoLog,binLog日志
3. 慢

### DROP
删光，最快

### TRUNCATE
1. 相当于DROP + CREATE  删除了再建一张表
2. 可以重置自增id

## 31.MySQL 中 INNER JOIN、LEFT JOIN 和 RIGHT JOIN 的区别是什么？
1. 连接条件一个表代表一个集合，比如id是连接条件，则用户表和订单的id作为一个集合
2. 内连接：取交集   左连接：包含左表的id的所有行，右表没有的返回null
3. 表join 两个表的所有列合在一起
```sql
-- 假设有用户表 users 和订单表 orders
-- users: id=1,2,3   orders: user_id=1,1,2

-- INNER JOIN：只返回有订单的用户
SELECT * FROM users u
INNER JOIN orders o ON u.id = o.user_id;
-- 结果：用户1（2条）、用户2（1条），用户3没订单不返回

-- LEFT JOIN：返回所有用户，没订单的也要
SELECT * FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- 结果：用户1、2有订单，用户3的 order 字段是 NULL

-- RIGHT JOIN：返回所有订单，关联不上用户的也要
SELECT * FROM orders o
RIGHT JOIN users u ON o.user_id = u.id;
-- 和上面 LEFT JOIN 结果一样，只是写法不同
```
### on 和 where 的区别
1. on是在合并过程中，拿一行 A.id ，去 B 中找：满足 A.id = B.user_id AND B.status = 1 的行，找到的行，返回数据；找不到的行，所有在b表中的数据填null进行返回

2. where先jion,再按where条件进行过滤

3. LEFT JOIN + WHERE =  INNER JOIN

例子：
表A
id | name
---------
1  | Tom
2  | Jack
3  | Lucy
表B
user_id | order | status
-----------------------
1       | A001  | 1
2       | A002  | 0

```sql
--on
SELECT *
FROM A
LEFT JOIN B
ON A.id = B.user_id AND B.status = 1;
```
结果：
id | name | order | status
--------------------------------
1  | Tom  | A001  | 1
2  | Jack | NULL  | NULL
3  | Lucy | NULL  | NULL
```sql
--where
SELECT *
FROM A
LEFT JOIN B ON A.id = B.user_id
WHERE B.status = 1;
```
结果：
先join
id | name | order | status
--------------------------------
1  | Tom  | A001  | 1
2  | Jack | A002  | 0
3  | Lucy | NULL  | NULL

再where过滤
id | name | order | status
----------------------------
1  | Tom  | A001  | 1

## 32.MySQL 中 DATETIME 和 TIMESTAMP 类型的区别是什么？
1. DATETIME存字符串，写什么存什么
2. TIMESTAMP存UTC，读出来会根据时区发生改变。存和取的时区都是看session时区。（多个时区下session不可控）
3. TIMESTAMP存在范围限制 1970 - 2038
4. 大部分场景用DATETIME  单一时区可用TIMESTAMP  复杂时区用DATETIME，在应用层做存和取的时区转换

## 33.三大范式
1. 第一范式 1NF：
每个字段必须是原子的，不能再拆分。
比如「地址」字段存「北京市朝阳区xxx路」就不符合，应该拆成省、市、区、详细地址四个字段。
2. 第二范式 2NF：
在 1NF 基础上，非主键字段必须完全依赖主键，不能只依赖主键的一部分。
这主要针对联合主键的场景，比如订单明细表用「订单ID + 商品ID」做联合主键，如果把「订单时间」也放进来就违反了 2NF，因为订单时间只依赖订单ID。
3. 第三范式 3NF：
在 2NF 基础上，非主键字段之间不能有依赖关系。
比如员工表里存了部门ID和部门名称，部门名称依赖部门ID而不是员工ID，这就是传递依赖，应该把部门名称拆到部门表里。

### 总结
1NF：字段不可再拆（原子性）
2NF：非主键必须完全依赖主键
3NF：非主键之间不能互相依赖（消除传递依赖）

## 34.在 MySQL 中，你使用过哪些函数？
### 字符串函数
```sql
-- 1 拼接
SELECT CONCAT('Hello', ' ', 'World');
-- Hello World

-- 2️ 子串
SELECT SUBSTRING('abcdef', 1, 3);
-- abc

-- 3 长度
SELECT LENGTH('abc');       -- 字节长度
SELECT CHAR_LENGTH('abc');  -- 字符长度

-- 4 去空格
SELECT TRIM('  hello  ');

-- 5 替换
SELECT REPLACE('hello world', 'world', 'mysql');

-- 6 大小写
SELECT LOWER('ABC');
SELECT UPPER('abc');
```
### 数值函数
```sql
-- 1 四舍五入
SELECT ROUND(3.1415, 2);  -- 3.14

-- 2 向上 / 向下取整
SELECT CEIL(3.1);   -- 4
SELECT FLOOR(3.9);  -- 3

-- 3 绝对值
SELECT ABS(-10);  -- 10

-- 4 随机数
SELECT RAND();
```
### 日期函数
```sql
--1 当前时间
SELECT NOW();        -- 当前时间
SELECT CURDATE();    -- 当前日期
SELECT CURTIME();    -- 当前时间（时分秒）

--2 日期加减
SELECT DATE_ADD(NOW(), INTERVAL 1 DAY);
SELECT DATE_SUB(NOW(), INTERVAL 1 DAY);

--3 计算时间差
SELECT DATEDIFF('2026-06-22', '2026-06-20'); -- 2天

--4 格式化时间
SELECT DATE_FORMAT(NOW(), '%Y-%m-%d %H:%i:%s');

--5 提取时间字段
SELECT YEAR(NOW());
SELECT MONTH(NOW());
SELECT DAY(NOW());
```
### 聚合函数
```sql
--1 COUNT
SELECT COUNT(*) FROM user;

--2 SUM
SELECT SUM(amount) FROM orders;

--3 AVG
SELECT AVG(score) FROM student;

--4 MAX / MIN
SELECT MAX(price), MIN(price) FROM goods;

--5 GROUP_CONCAT
SELECT class, GROUP_CONCAT(name)
FROM student
GROUP BY class;
--把多行数据拼接成一行字符串
--按class分组，所以name列展示在class一行

--GROUP BY name = 把相同 name 的数据分组，用于统计或去重
```
### 流程控制函数
```sql
--1 IF
SELECT IF(score > 60, '及格', '不及格') FROM student;

--2 CASE
SELECT
  CASE 
    WHEN score >= 90 THEN '优秀'
    WHEN score >= 60 THEN '及格'
    ELSE '不及格'
  END
FROM student;
--3 IFNULL
SELECT IFNULL(score, 0) FROM student;
-- 如果为 NULL → 用默认值
```
### 窗口函数
窗口函数是在“保留原始行”的情况下，对数据进行分组计算的函数
```sql
函数() OVER (
    PARTITION BY 分组字段
    ORDER BY 排序字段
)

--1 ROW_NUMBER（行号）
SELECT name, score,
       ROW_NUMBER() OVER (ORDER BY score DESC) AS rn
FROM student;
--每行按 score 排序，rn列1,2,3,4...（没有重复）

--2 RANK（排名）
SELECT name, score,
       RANK() OVER (ORDER BY score DESC)
FROM student;
-- 并列名次相同，下一名跳号

--3 DENSE_RANK（排名 不跳号）
SELECT name, score,
       DENSE_RANK() OVER (ORDER BY score DESC)
FROM student;

--4 SUM / AVG（窗口聚合）
SELECT name, class, score,
       AVG(score) OVER (PARTITION BY class)
FROM student;
-- 每个班级平均分（但保留每行）

--5 累加（加上当前行）
SELECT name, score,
       SUM(score) OVER (ORDER BY score)
FROM student;

--6 LAG / LEAD（前后行对比)
SELECT name, score,
       LAG(score) OVER (ORDER BY score)
FROM student; --上一行

SELECT name, score,
       LEAD(score) OVER (ORDER BY score)
FROM student; --下一行

--例：每个班级前2名
SELECT *
FROM (
    SELECT *,
           ROW_NUMBER() OVER (PARTITION BY class ORDER BY score DESC) AS rn
    FROM student
) t
WHERE rn <= 2;

--PARTITION BY（分组窗口）
相同的class单独排名
| name | class | score | rk |
| ---- | ----- | ----- | -- |
| A    | 1     | 100   | 1  |
| B    | 1     | 100   | 1  |
| C    | 1     | 90    | 3  |
| D    | 2     | 95    | 1  |
| E    | 2     | 80    | 2  |
```

## 35.数据类型
### text
1. 存在外部页，多io
2. 存不确定长度的数据，如文章描述
### varchar(n)
1. 变长 长度标识（1（<=255字节）或2字节） + 实际数据
2. 前缀索引：如果字符串太长可取前面几个字段作为索引
3. n代表字符，限制大小按字节算  
4. 排序，分组，创建临时表会按 字段定义的最大值 分配空间，所以要定得合理
### 主键索引
1. 自增索引可直接用bigint，int满了会报错
2. 可用分布式id作为主键
### 支付金额
1. BIGINT
- 一般存到分 10.23元 → 存 1023
- 比DECIMAL快，存储空间小
- java里面对应用long
1. DECIMAL
- DECIMAL(M,D) M是总位数，D是小数位数
- 普通金额用DECIMAL(18,2)，费率计算DECIMAL(18,4)
- java里面对应用BigDecimal
```java
// 创建 BigDecimal 禁止用 double 构造器
BigDecimal bad = new BigDecimal(0.1);   // 精度已经丢了
BigDecimal good = new BigDecimal("0.1"); // 字符串构造，精确

// 除法必须指定精度和舍入模式
BigDecimal result = a.divide(b, 2, RoundingMode.HALF_UP);

// 比较用 compareTo，不要用 equals
// equals 会比较精度，1.0 和 1.00 不相等
if (a.compareTo(b) == 0) { ... }
```
### 大音视频
1. BLOB = Binary Large Object （二进制大对象）
2. 存的是“二进制数据”
3. 行内存指针 ， 真实数据存在行外页  ， 意味着更多io
4. 单一个数据太大，影响bufferPool,binlog（insert长长的二进制数据）
5. 推荐阿里云OSS等

### 视图
1. 相当于带名字的select语句
2. 不存真实数据，调用视图相当于调用创建视图的select语句
3. 用于复杂报表查询，权限控制（视图中只展示可展示的列）
4. 满足1.基于单表，没有join 2. 没有聚合函数 3.没有子查询 4.没有表达式和计算列 可更新（insert,update,delete）视图里的数据，一般视图作为只读
5. 分布式用不到

### 游标
1. 用于个性化处理每行数据
2. mysql搭配存储过程
3. 性能差
```sql
-- 修改分隔符  默认 SQL 结束符是 ; 存储过程内部也有 ; 会冲突 
DELIMITER //
CREATE PROCEDURE adjust_salary()
-- 存储过程的代码开始
BEGIN
    -- 声明变量
    DECLARE done INT DEFAULT FALSE;
    DECLARE emp_id INT;
    DECLARE emp_salary DECIMAL(10,2);

    -- 1.声明游标，指定要遍历的结果集
    DECLARE salary_cursor CURSOR FOR
        SELECT id, salary FROM employees WHERE department = 'tech';
    -- 遍历结束时的处理器：当 FETCH 没有数据时（读取到末尾），自动执行done = TRUE ，防止游标读完数据 报 NOT FOUND
    DECLARE CONTINUE HANDLER FOR NOT FOUND SET done = TRUE;
    --2.打开游标
    OPEN salary_cursor;
    --定义一个循环（名字叫 read_loop）
    read_loop: LOOP
        -- 3.从游标取一行数据
        FETCH salary_cursor INTO emp_id, emp_salary;

        --如果已经没有数据了，跳出循环
        IF done THEN
            LEAVE read_loop;
        END IF;

        -- 对每一行做业务逻辑，比如涨薪 10%
        UPDATE employees SET salary = emp_salary * 1.1 WHERE id = emp_id;
    END LOOP;
    -- 4.关闭游标
    CLOSE salary_cursor;
END //
DELIMITER ;
```
## 36.在什么情况下，不推荐为数据库建立索引？
1. 数据量很小的表（千百）
2. 写入频繁的表
3. 区分度低的字段（男女）
4. 查询操作不会用到的字段
5. 大字段

### 索引的维护成本
1. insert  涉及到主键，要更新所有索引树
2. update  涉及到的索引要更新索引树，如果涉及主键全要更新（二级索引树里存主键），先删除原来索引再插入新的
3. delete  标记删除，后续清理

## 37.MySQL 中 EXISTS 和 IN 的区别是什么？
```sql
-- EXISTS 写法：外表 mianshiya 小，子查询 order 大
-- 用外表m一条条去子表里查， 找到就停止
SELECT * FROM mianshiya m
WHERE EXISTS (SELECT 1 FROM `order` o WHERE m.user_id = o.user_id);

-- IN 写法：子查询 order 小，外表 mianshiya 大
-- 先执行子查询内表，把结果放到临时集合里，再去和 外表 比较
SELECT * FROM mianshiya
WHERE user_id IN (SELECT user_id FROM `order`);
```
- in碰上为null的数据会有问题
```sql
-- 假设 order 表有一行 user_id = NULL  查询外表不在内表的id
SELECT * FROM mianshiya WHERE user_id NOT IN (SELECT user_id FROM `order`);
-- 结果：空！一行都没有  
-- 因为外表的id比较内表的null时，返回的是unknown，而not in需要都返回false，所以外表id一个都查不到
```

## 38.如何实现数据库的不停服迁移？
1. 建从库作为新库
2. 双写：从库和新库一起更新  （事务一致性，幂等性，先写旧库，同步延迟）
3. 灰度切流：从主要读旧库慢慢切换到读新库

## 39.MySQL 数据库的性能优化方法有哪些？
1. SQL 优化  select*  limit 1000,10
2. 索引优化   联合索引，索引失效
3. 表结构设计  合适的类型和大小  主键  分表
4. 架构优化   读写分离 分库分表  redis缓存
5. 参数与系统优化  缓存池参数   binlog   慢查询日志

## 40.MySQL 的查询优化器如何选择执行计划？
1. 解析阶段 -> 预处理阶段 -> 优化阶段
2. 优化阶段会计算成本，成本：I/O成本（处理多少页）和CPU成本（读多少行）


## 41.逻辑删除
问题：唯一主键可能发生重复  比如 email为唯一索引 新加 - 删除（改is_delete）- 再加 重复的email不满足唯一性

1. is_delete不用0，1  删除的改成时间或者主键id填充
2. 反复修改同一条记录，但需要  流水表  记录操作

## 42.什么是数据库的逻辑外键？数据库的物理外键和逻辑外键各有什么优缺点？
1. 逻辑外键：应用层代码保证表与表之间的外键关系，数据库里面不建外键    优选
2. 物理外键：数据库明确写FOREIGN KEY (user_id) REFERENCES user(id)

## 43.MySQL 事务的二阶段提交是什么？
1. 保证 binlog 与 InnoDB redo log 一致性
2. prepare阶段：innodb层写redolog,标记为prepare
3. commit阶段：server层把操作写到binlog,binlog成功后把redolog标记为commit
### 为什么要有
1. redo log（InnoDB）用于崩溃恢复 ；binlog（Server层）用于主从复制
2. redo成功 / binlog失败，事务提交了，从库没数据
3. binlog成功 / redo失败，事务没提交回滚了，从库有数据主库没有
### 为什么可以解决
发生宕机，redo = prepare，检查 binlog 写没写，如果写了则说明redolog还没来得及提交，直接提交；没有写则redo 没 commit，可以直接回滚
https://pic.code-nav.cn/mianshiya/question_picture/1783388929455529986/yYl3cAym_image_mianshiya.webp

## 44.MySQL 三层 B+ 树能存多少数据？
2000万
1. B+树一个节点就是一页 16kb
2. 前两层存索引  一个索引由主键（8b）和指针(6b) 14字节  16kb / 14 b = 1170 个 也就是一页可以存1170个索引   两层一共索引 1170 * 1170 
3. 最后一个页存数据 一条数据1kb 一个页存16条数据 
4. 三层 B+ 树可以存 1170*1170*16 = 2190万条数据

## 45. 为什么 MySQL 索引用的是 B+ 树而不是红黑树？
1. IO次数少 每访问一个节点就是一次IO（查一条数据一个节点一个节点下来，就是一层一次IO）
2. B+树一般就三层
3. 红黑树每个节点只能2 个子节点，千万条数据有20几层
4. B+树有序双向链表
5. B+树非叶子节点不存数据，B树和红黑树每个节点都存数据
6. 内存适合红黑树

## 46.SQL 中 select、from、join、where、group by、having、order by、limit 的执行顺序是什么？
select在having后面
1. from 哪张表
2. join 多表组合
3. where 过滤行
4. group by 分组
5. having 对分组结果过滤
6. select 选择列
7. order by 排序
8. limit 限制条数

## 47.什么是 CDC（Change Data Capture）？常见的 CDC 工具有哪些？
1. CDC（Change Data Capture）是一种用于捕获数据库中数据变化（如插入、更新、删除),然后进行同步到其他系统如（Kafka，Elasticsearch，数据仓库）
2. 常见实现方式轮询、触发器 以及 基于数据库日志（如 MySQL binlog）的实现
3. 常见的 CDC 工具有 Canal、Debezium、Maxwell 和 Flink CD

## 48.多线程并发同步数据时（数据库的数据同步到数仓中）需要注意什么问题？
在多线程并发同步数据过程中，需要重点关注顺序性、幂等性、事务一致性以及数据丢失或重复等问题。通常需要保证同一数据按照顺序处理，可以通过按主键分片实现；同时通过幂等机制避免重复数据问题；对于事务，要保证整体同步而不是拆分执行。此外，需要结合 offset 管理和重试机制来避免数据丢失，并通过合理的并发控制来避免更新冲突。
MySQL
↓
binlog
↓
CDC（Canal / Flink CDC）
↓
Kafka（分区保证顺序）
↓
Flink / 消费程序
↓
数仓（ODS / DWD）




数据结构，redo log流程图，RR与幻读，MySQL 二级索引有 MVCC 快照吗？，	如果 MySQL 中没有 MVCC，会有什么影响？，	MySQL 默认的事务隔离级别是什么？为什么选择这个级别？，explain
业务流程：
电子签
## 临时设置环境变量
$env:JIB_USERNAME="clu57"
$env:JIB_PASSWORD="x"
