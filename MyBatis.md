# 入门
## 依赖
```xml
<dependency>
  <groupId>org.mybatis</groupId>
  <artifactId>mybatis</artifactId>
  <version>x.x.x</version>
</dependency>
```
## xml构建SqlSessionFactory
- configuration（配置）
  - properties（属性）
  - settings（设置）
  - typeAliases（类型别名）
  - typeHandlers（类型处理器）
  - objectFactory（对象工厂）
  - plugins（插件）
  - environments（环境配置）
    - environment（环境变量）
        - transactionManager（事务管理器）
        - dataSource（数据源）
  - databaseIdProvider（数据库厂商标识）
  - mappers（映射器）
```xml
<!-- xml文件声明 -->
<?xml version="1.0" encoding="UTF-8" ?>

<!--引入 MyBatis 的 DTD，用于格式校验  -->
<!DOCTYPE configuration
 PUBLIC "-//mybatis.org//DTD Config 3.0//EN"
  "https://mybatis.org/dtd/mybatis-3-config.dtd">

<!-- 配置标签 -->
<configuration>

<!-- ===========================  属性配置properties  ========================================== -->
<!-- 读取外部配置文件 或者 url 二选一-->
<properties resource="org/mybatis/example/config.properties" url="http://www.hxstrive.com/data/database.properties" >
<!-- 内部属性配置 -->
  <property name="username" value="dev_user"/>
  <property name="password" value="F2Fa3!33TYyg"/>
</properties>

<!-- ===========================  MyBatis全局配置Settings  ========================================== -->
<settings>

    <!-- 全局缓存开关；影响所有 mapper 中已配置的二级缓存。默认：true -->
    <setting name="cacheEnabled" value="true"/>

    <!-- 延迟加载总开关；开启后关联对象延迟加载，可被每个关联的 fetchType 覆盖。默认：false -->
    <setting name="lazyLoadingEnabled" value="false"/>

    <!-- 激进延迟加载：开启后任意方法调用都会触发加载所有延迟属性；否则按需加载。默认：false（3.4.1 及之前为 true） -->
    <setting name="aggressiveLazyLoading" value="false"/>

    <!-- 已废弃：multipleResultSetsEnabled；该选项已无实际效果。默认：true -->
    <setting name="multipleResultSetsEnabled" value="true"/>

    <!-- 使用列标签而非列名（依赖驱动实现，建议按驱动表现测试）。默认：true -->
    <setting name="useColumnLabel" value="true"/>

    <!-- 允许 JDBC 自动生成主键（需驱动支持），为 true 时强制使用。默认：false -->
    <setting name="useGeneratedKeys" value="false"/>

    <!-- 自动映射行为：NONE / PARTIAL / FULL；默认：PARTIAL -->
    <setting name="autoMappingBehavior" value="PARTIAL"/>

    <!-- 当自动映射遇到未知列或类型时的行为：NONE / WARNING / FAILING；默认：NONE -->
    <setting name="autoMappingUnknownColumnBehavior" value="NONE"/>

    <!-- 默认执行器类型：SIMPLE / REUSE / BATCH；默认：SIMPLE -->
    <setting name="defaultExecutorType" value="SIMPLE"/>

    <!-- 语句默认超时（秒）；未设置为 null。默认：未设置 -->
    <!-- <setting name="defaultStatementTimeout" value="30"/> -->

    <!-- 结果集 fetchSize 的建议值；可在查询级别覆盖。默认：未设置 -->
    <!-- <setting name="defaultFetchSize" value="1000"/> -->

    <!-- 结果集滚动类型：FORWARD_ONLY / SCROLL_SENSITIVE / SCROLL_INSENSITIVE / DEFAULT；默认：未设置 -->
    <!-- <setting name="defaultResultSetType" value="DEFAULT"/> -->

    <!-- 在嵌套语句中是否允许 RowBounds 分页：允许则设为 false。默认：false -->
    <setting name="safeRowBoundsEnabled" value="false"/>

    <!-- 在嵌套语句中是否允许 ResultHandler：允许则设为 false。默认：true -->
    <setting name="safeResultHandlerEnabled" value="true"/>

    <!-- 下划线转驼峰自动映射：A_COLUMN -> aColumn。默认：false -->
    <setting name="mapUnderscoreToCamelCase" value="false"/>

    <!-- 本地缓存作用域：SESSION / STATEMENT；默认：SESSION -->
    <setting name="localCacheScope" value="SESSION"/>

    <!-- 空值的 JDBC 类型默认指定；常见：NULL / VARCHAR / OTHER。默认：OTHER -->
    <setting name="jdbcTypeForNull" value="OTHER"/>

    <!-- 触发延迟加载的方法列表（逗号分隔）。默认：equals,clone,hashCode,toString -->
    <setting name="lazyLoadTriggerMethods" value="equals,clone,hashCode,toString"/>

    <!-- 动态 SQL 默认脚本语言；默认：org.apache.ibatis.scripting.xmltags.XMLLanguageDriver -->
    <setting name="defaultScriptingLanguage" value="org.apache.ibatis.scripting.xmltags.XMLLanguageDriver"/>

    <!-- Enum 默认 TypeHandler；默认：org.apache.ibatis.type.EnumTypeHandler（新增于 3.4.5） -->
    <setting name="defaultEnumTypeHandler" value="org.apache.ibatis.type.EnumTypeHandler"/>

    <!-- 当结果集中值为 null 时是否调用 setter/put（基本类型无法设为 null）。默认：false -->
    <setting name="callSettersOnNulls" value="false"/>

    <!-- 当一行所有列为空时是否返回空实例而非 null；也影响嵌套结果集。默认：false（新增于 3.4.2） -->
    <setting name="returnInstanceForEmptyRow" value="false"/>

    <!-- 日志前缀；默认：未设置 -->
    <!-- <setting name="logPrefix" value="[MyBatis]"/> -->

    <!-- 日志实现：SLF4J / LOG4J(3.5.9 起废弃) / LOG4J2 / JDK_LOGGING / COMMONS_LOGGING / STDOUT_LOGGING / NO_LOGGING；默认：未设置 -->
    <!-- <setting name="logImpl" value="SLF4J"/> -->

    <!-- 延迟加载代理工具：CGLIB(3.5.10 起废弃) / JAVASSIST；默认：JAVASSIST -->
    <setting name="proxyFactory" value="JAVASSIST"/>

    <!-- VFS 实现（逗号分隔的全限定类名）。默认：未设置 -->
    <!-- <setting name="vfsImpl" value="org.apache.ibatis.io.JBoss6VFS"/> -->

    <!-- 允许使用方法签名中的名称作为参数名（需 Java 8 且 -parameters）。默认：true（新增于 3.4.1） -->
    <setting name="useActualParamName" value="true"/>

    <!-- 指定提供 Configuration 的工厂类（需包含 static Configuration getConfiguration()）。默认：未设置（新增于 3.2.3） -->
    <!-- <setting name="configurationFactory" value="com.example.mybatis.ConfigurationFactory"/> -->

    <!-- 删除 SQL 中多余空格（会影响字符串字面量）；默认：false（新增于 3.5.5） -->
    <setting name="shrinkWhitespacesInSql" value="false"/>

    <!-- 默认 sql provider 类（用于注解省略 type/value 时）。默认：未设置（新增于 3.5.6） -->
    <!-- <setting name="defaultSqlProviderType" value="com.example.mybatis.SqlProvider"/> -->

    <!-- foreach 标签的 nullable 默认值。默认：false（新增于 3.5.9） -->
    <setting name="nullableOnForEach" value="false"/>

    <!-- 构造器自动映射时按参数名匹配列，而非列顺序。默认：false（新增于 3.5.10） -->
    <setting name="argNameBasedConstructorAutoMapping" value="false"/>

  </settings>

<!-- ===========================  MyBatis别名配置typeAliases  ========================================== -->
<typeAliases>
<!-- 简化全类名，通过alias取别名 -->
  <typeAlias alias="Author" type="domain.blog.Author"/>
  <typeAlias alias="Blog" type="domain.blog.Blog"/>
  <typeAlias alias="Comment" type="domain.blog.Comment"/>
  <typeAlias alias="Post" type="domain.blog.Post"/>
  <typeAlias alias="Section" type="domain.blog.Section"/>
  <typeAlias alias="Tag" type="domain.blog.Tag"/>
</typeAliases>
<!-- 指定包名，若无注解，该包下所有类用类名小写起别名 -->
<typeAliases>
  <package name="domain.blog"/>
</typeAliases>
<!--
//有注解注解优先
@Alias("author")
public class Author {
    ...
}
  -->
<!-- 默认别名 -->
<!-- 别名	映射的类型
_byte	byte
_long	long
_short	short
_int	int
_integer	int
_double	double
_float	float
_boolean	boolean
string	String
byte	Byte
long	Long
short	Short
int	Integer
integer	Integer
double	Double
float	Float
boolean	Boolean
date	Date
decimal	BigDecimal
bigdecimal	BigDecimal
object	Object
map	Map
hashmap	HashMap
list	List
arraylist	ArrayList
collection	Collection
iterator	Iterator -->

<!-- ===========================      todolist  ========================================== -->
<!-- ===========================  MyBatis类处理器typeHandlers  ========================================== -->
    <typeHandlers>
        <!-- 单个注册 -->
        <typeHandler handler="com.example.handler.OrderStatusTypeHandler"/>
        <!-- 批量注册包下所有类型处理器 -->
        <package name="com.example.handler"/>
    </typeHandlers>

<!-- ===========================  MyBatis对象工厂objectFactory  ========================================== -->
<!-- ===========================  MyBatis插件plugins  ========================================== -->
<!-- ===========================      todolist  ========================================== -->

<!-- ===========================  MyBatis环境environments  ========================================== -->
<!-- 配置多个环境下的mybatis，设置默认环境efault="development"，每个数据库对应一个 SqlSessionFactory 实例 -->
  <environments default="development">
  <!-- dev环境下的配置 -->
    <environment id="development">

    <!-- ===========================  MyBatis事务管理器transactionManager  =========================== -->
    <!-- 事务管理器-单标签 -->
      <transactionManager type="JDBC"/>
    <!-- 事务管理器-JDBC -->
    <transactionManager type="JDBC">
    <!-- 关闭自动提交事务 -->
      <property name="skipSetAutoCommitOnClose" value="true"/>
    </transactionManager>
     <!-- 事务管理器-MANAGED -->  
    <transactionManager type="MANAGED">
        <!-- 可选属性：是否关闭连接（默认 false，由容器比如Spring的org.mybatis.spring.transaction.SpringManagedTransactionFactory管理） -->
        <property name="closeConnection" value="false"/>
    </transactionManager>
    <!-- ===========================  MyBatis数据源dataSource  =========================== -->
      <dataSource type="POOLED">
      
      <!-- 先读properties内的proerty,再读properties属性写的外部配置文件，最后读方法形参进行一一覆盖 -->
        <property name="driver" value="${driver}"/>
        <property name="url" value="${url}"/>

        <property name="username" value="${username}"/>
        <property name="password" value="${password}"/>
      </dataSource>
    </environment>
  </environments>
  <mappers>
    <mapper resource="org/mybatis/example/BlogMapper.xml"/>
  </mappers>
</configuration>
```
```java
String resource = "org/mybatis/example/mybatis-config.xml";
InputStream inputStream = Resources.getResourceAsStream(resource);
SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(inputStream);


/* ===========================  MyBatis类处理器typeHandlers(存疑，需要JDBC基础)  ========================================== */
// 订单状态枚举
public enum OrderStatus {
    UNPAID(1, "未支付"),
    PAID(2, "已支付"),
    CANCELLED(3, "已取消");

    private int code;
    private String desc;

    OrderStatus(int code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    // 根据编码获取枚举
    public static OrderStatus getByCode(int code) {
      //枚举类静态方法values()，用于获取枚举类的数组 [UNPAID,PAID,CANCELLED]
        for (OrderStatus status : values()) {
            if (status.code == code) {
                return status;
            }
        }
        throw new IllegalArgumentException("无效的订单状态编码：" + code);
    }

    // getter/setter
    public int getCode() { return code; }
    public String getDesc() { return desc; }
}

// 自定义类型处理器  实现 org.apache.ibatis.type.TypeHandler 接口，或继承org.apache.ibatis.type.BaseTypeHandler
public class OrderStatusTypeHandler extends BaseTypeHandler<OrderStatus> {
    // 设置参数（Java 类型 → JDBC 类型）
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, OrderStatus parameter, JdbcType jdbcType) throws SQLException {
        ps.setInt(i, parameter.getCode()); // 将枚举编码存入数据库
    }

    // 从结果集按列名获取值（JDBC 类型 → Java 类型）
    @Override
    public OrderStatus getNullableResult(ResultSet rs, String columnName) throws SQLException {
        int code = rs.getInt(columnName);
        return rs.wasNull() ? null : OrderStatus.getByCode(code);
    }

    // 从结果集按列索引获取值
    @Override
    public OrderStatus getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        int code = rs.getInt(columnIndex);
        return rs.wasNull() ? null : OrderStatus.getByCode(code);
    }

    // 从存储过程结果获取值
    @Override
    public OrderStatus getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        int code = cs.getInt(columnIndex);
        return cs.wasNull() ? null : OrderStatus.getByCode(code);
    }
}
```
