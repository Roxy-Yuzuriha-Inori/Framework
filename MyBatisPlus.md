# 实体类命名
- DTO 前端给后端
- VO  后端给前端
- PO/DO 数据库对应实体类

# 配置
```yml
# application.yml
mybatis-plus:
  global-config:
    db-config:
      # 逻辑删除全局配置
      logic-delete-field: isDeleted # 全局逻辑删除字段名（实体类字段名）
      logic-not-delete-value: 0 # 未删除值
      logic-delete-value: 1 # 已删除值
      # 枚举字段类型匹配（可选，默认自动识别）
      enum-type-handler: com.baomidou.mybatisplus.extension.handlers.MybatisEnumTypeHandler
      type-handlers: com.baomidou.mybatisplus.extension.handlers.JacksonTypeHandler
  configuration:
    default-executor-type: batch # 开启MyBatis批处理执行器
    # 枚举类型处理器（核心）
    type-handlers-package: com.baomidou.mybatisplus.extension.handlers

```
# 依赖
```xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.5</version> 
</dependency>
```
# 继承BaseMapper

## po实体
```java
// PO实体类（与数据库表 user 映射）
@Data
@TableName("user") // 表名不一致时指定
public class User {
    @TableId(type = IdType.AUTO) // 主键自增
    private Long id;
    private String username;
    private Integer age;
    private String email;
    @TableLogic // 逻辑删除标记
    private Integer isDeleted;
}

// Mapper 接口（核心：继承 BaseMapper）
public interface UserMapper extends BaseMapper<User> {
    // 基础CRUD无需手写，直接继承
}
```
### @TableId
- 作用于主键id

| 策略类型 | 说明 | 适用场景 |
|---------|------|----------|
| `IdType.AUTO` | 数据库自增（依赖数据库主键自增配置） | 单表主键、MySQL 自增主键 |
| `IdType.NONE` | 无策略：需手动设置主键值，MP 不干预 | 主键由业务代码生成（如自定义编号） |
| `IdType.INPUT` | 手动输入：与 NONE 类似，明确表示主键需手动赋值 | 主键为业务唯一标识（如手机号、订单号） |
| `IdType.ASSIGN_ID` | MP 自动生成雪花算法 ID（Long 类型），无需数据库自增 | 分布式系统、高并发、分表分库 |
| `IdType.ASSIGN_UUID` | MP 自动生成 UUID（String 类型，去掉横线） | 主键为字符串、无需有序生成 |
## ServiceImpl基本方法查询
```java
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    // 示例1：根据ID查询
    public User getById(Long id) {
        return userMapper.selectById(id);
    }

    // 示例2：批量查询
    public List<User> listByIds(List<Long> ids) {
        return userMapper.selectBatchIds(ids);
    }
}
```
## ServiceImpl where条件查询
```java
//queryWrapper构建条件
@Service
public class UserServiceImpl implements UserService {
    @Autowired
    private UserMapper userMapper;

    // 示例：查询年龄>20 且 邮箱包含@qq.com 的用户
    public List<User> listUserByCondition() {
        // 1. 创建条件构造器
        QueryWrapper<User> queryWrapper = new QueryWrapper<>();
        // 2. 链式构建条件
        queryWrapper.gt("age", 20) // gt = greater than（>）
                   .like("email", "@qq.com") // like 模糊查询
                   .orderByDesc("id"); // 按ID降序
        // 3. 调用Mapper方法（传入Wrapper）
        return userMapper.selectList(queryWrapper);
    }
}

//lambdaWrapper构建条件（推荐）
// 示例：查询年龄>=18 且 用户名不为空的用户
public List<User> listUserByLambda() {
    LambdaQueryWrapper<User> lambdaWrapper = new LambdaQueryWrapper<>();
    lambdaWrapper.ge(User::getAge, 18) // ge = greater than or equal（>=）
                .isNotNull(User::getUsername)
                .eq(User::getIsDeleted, 0); // eq = equal（=）
    return userMapper.selectList(lambdaWrapper);
}
```
## Wrapper 常用链式方法说明
| 方法 | 作用 | 示例 |
|------|------|------|
| `eq(R column, Object val)` | 等于 = | `eq("age", 20)` / `eq(User::getAge, 20)` |
| `ne(R column, Object val)` | 不等于 != | `ne("email", null)` |
| `gt(R column, Object val)` | 大于 > | `gt("age", 18)` |
| `ge(R column, Object val)` | 大于等于 >= | `ge("age", 18)` |
| `lt(R column, Object val)` | 小于 < | `lt("age", 30)` |
| `le(R column, Object val)` | 小于等于 <= | `le("age", 30)` |
| `like(R column, Object val)` | 模糊查询 LIKE | `like("username", "张")`（%张%） |
| `likeLeft(R column, Object val)` | 左模糊 LIKE | `likeLeft("username", "张")`（%张） |
| `likeRight(R column, Object val)` | 右模糊 LIKE | `likeRight("username", "张")`（张%） |
| `isNull(R column)` | 字段为 NULL | `isNull("email")` |
| `isNotNull(R column)` | 字段不为 NULL | `isNotNull("username")` |
| `in(R column, Collection<?> coll)` | IN 查询 | `in("age", Arrays.asList(18, 20, 22))` |
| `notIn(R column, Collection<?> coll)` | NOT IN 查询 | `notIn("id", 1, 2, 3)` |
| `between(R column, Object val1, Object val2)` | BETWEEN ... AND ... | `between("age", 18, 30)` |
| `notBetween(R column, Object val1, Object val2)` | NOT BETWEEN ... AND ... | `notBetween("age", 18, 30)` |
| `orderByAsc(R... columns)` | 升序排序 | `orderByAsc("age", "id")` |
| `orderByDesc(R... columns)` | 降序排序 | `orderByDesc(User::getId)` |
| `groupBy(R... columns)` | 分组 GROUP BY | `groupBy("age", "gender")` |
| `having(String sqlHaving, Object... params)` | 分组后筛选 HAVING | `having("COUNT(*) > 1")` |
| `and(Consumer<Wrapper<T>> consumer)` | 拼接 AND 条件 | `and(w -> w.eq("gender", 1).gt("age", 20))` |
| `or(Consumer<Wrapper<T>> consumer)` | 拼接 OR 条件 | `or(w -> w.eq("email", "xxx@qq.com").eq("phone", "138xxxx"))` |
| `nested(Consumer<Wrapper<T>> consumer)` | 嵌套条件（括号包裹） | `nested(w -> w.eq("a", 1).or().eq("b", 2))` |
| `select(R... columns)` | 指定查询字段 | `select("id", "username")` / `select(User::getId, User::getUsername)` |

## Mapper层复杂SQL
- 注解方式
```java
// UserMapper 接口
public interface UserMapper extends BaseMapper<User> {
    // 自定义SQL：多字段查询 + 复用Wrapper条件
    @Select("SELECT u.id, u.username, u.age, d.dept_name " +
            "FROM user u LEFT JOIN dept d ON u.dept_id = d.id " +
            "${ew.customSqlSegment}") // 引用Wrapper条件
    List<Map<String, Object>> selectUserDept(@Param(Constants.WRAPPER) QueryWrapper<User> wrapper);
}

// Service 调用
public List<Map<String, Object>> selectUserDeptByAge(Integer age) {
    QueryWrapper<User> wrapper = new QueryWrapper<>();
    wrapper.gt("u.age", age) // 注意表别名
           .eq("d.dept_name", "研发部");
    return userMapper.selectUserDept(wrapper);
}
```
- XML方式
```java
//Service调用
public List<User> selectUserByCustomSql(Integer minAge) {
    LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<>();
    wrapper.ge(User::getAge, minAge)
           .isNotNull(User::getEmail)
           .orderByDesc(User::getCreateTime);
    return userMapper.selectUserByCustomSql(wrapper);
}

//Mapper 接口定义方法
public interface UserMapper extends BaseMapper<User> {
    // XML自定义SQL，复用Wrapper
    List<User> selectUserByCustomSql(@Param(Constants.WRAPPER) LambdaQueryWrapper<User> wrapper); //Constants.WRAPPER等同于"ew"
}

```
```xml
<!-- Mapper对应xml文件 -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" 
"http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">
    <select id="selectUserByCustomSql" resultType="com.example.entity.User">
        SELECT id, username, age, email 
        FROM user 
        ${ew.customSqlSegment} <!-- 引用Wrapper的WHERE条件 -->
    </select>
</mapper>

```
# 继承IService
```java
//----------------------------Controller调Service----------------------------
// com.example.controller.UserController
import com.example.entity.User;
import com.example.service.UserService;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import java.util.List;

@RestController
public class UserController {
    @Autowired
    private UserService userService;

    // 调用IService内置方法
    @GetMapping("/list")
    public List<User> listAll() {
        return userService.list(); // IService 内置查询所有方法
    }

    // 调用自定义业务方法
    @GetMapping("/listByAge")
    public List<User> listByAge(Integer min, Integer max) {
        return userService.listByAgeRange(min, max);
    }
}

//----------------------------Service接口继承IService------------------------------------------
// com.example.service.UserService
import com.baomidou.mybatisplus.extension.service.IService;
import com.example.entity.User;

// 核心：继承 IService<T>，T 为实体类
public interface UserService extends IService<User> {
    // 自定义业务方法（非必须，可仅用IService内置方法）
    List<User> listByAgeRange(Integer minAge, Integer maxAge);
}

//----------------------------Service实现类----------------------------
// com.example.service.impl.UserServiceImpl
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import com.example.entity.User;
import com.example.mapper.UserMapper;
import com.example.service.UserService;
import org.springframework.stereotype.Service;

// 核心：继承 ServiceImpl<Mapper接口, 实体类> + 实现自定义Service接口
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    // 实现自定义业务方法（可直接用 baseMapper 调用Mapper方法）
    @Override
    public List<User> listByAgeRange(Integer minAge, Integer maxAge) {
        return lambdaQuery() // IService 内置链式查询（替代Wrapper）
                .ge(User::getAge, minAge)
                .le(User::getAge, maxAge)
                .list();
    }
}
```
## 批量插入
```java
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    // 批量新增用户
    public boolean batchAddUser(List<User> userList) {
        // 方式1：默认批次（1000条/批）
        // return saveBatch(userList);
        
        // 方式2：自定义批次（如500条/批，适合数据量大时）
        return saveBatch(userList, 500);
    }
}
```
# 转换器
## 枚举转换器
- 枚举类
```java
//枚举类
// 状态枚举（数据库存储 INT：0=禁用，1=启用）
public enum StatusEnum {
    DISABLE(0, "禁用"),
    ENABLE(1, "启用");

    // 与数据库映射的值（标记@EnumValue）
    @EnumValue
    private final Integer code;
    // 枚举描述（仅代码使用，不映射数据库）
    private final String desc;

    StatusEnum(Integer code, String desc) {
        this.code = code;
        this.desc = desc;
    }

    // 可选：添加反向解析方法（根据code获取枚举）
    public static StatusEnum getByCode(Integer code) {
        for (StatusEnum status : values()) {
            if (status.code.equals(code)) {
                return status;
            }
        }
        return null;
    }

    // getter
    public Integer getCode() { return code; }
    public String getDesc() { return desc; }
}

//数据库实体类
@Data
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    // 枚举类型字段（数据库存储INT）
    private StatusEnum status; 
}

```
## JSON转换器
- JSON类
```java
// 扩展属性类（对应JSON结构）
@Data
public class ExtInfo {
    private String hobby; // 爱好
    private List<String> tags; // 标签列表
    private Map<String, Object> params; // 扩展参数
}

//数据库实体类
@Data
@TableName("user")
public class User {
    @TableId(type = IdType.AUTO)
    private Long id;
    private String username;
    // JSON字段：指定JacksonTypeHandler处理器
    @TableField(typeHandler = JacksonTypeHandler.class)
    private ExtInfo extInfo; // 存储为JSON字符串
    // 也可直接用Map
    // @TableField(typeHandler = JacksonTypeHandler.class)
    // private Map<String, Object> extMap;
}
```
# TO自定义转换器，分页
自定义枚举处理器代码逐行详解
该自定义枚举处理器的核心是实现 MyBatis 提供的 BaseTypeHandler 接口，完成「数据库字符串（如 "启用"/"禁用"）」与「Java 枚举（StatusEnum）」之间的双向转换。以下是逐行拆解分析，帮助理解每个方法的作用、参数含义和设计逻辑。
一、核心前置知识
BaseTypeHandler<T>：MyBatis 提供的类型处理器基类，泛型 T 表示要转换的 Java 类型（此处为 StatusEnum），需重写 4 个核心方法实现双向转换。
作用场景：数据库存储枚举的「描述字符串」（如 "启用"/"禁用"），而非数字编码，MP 内置的 MybatisEnumTypeHandler 无法满足此需求，因此自定义处理器。
@Component 注解：将处理器交给 Spring 管理，确保 MyBatis 能扫描并加载（也可通过 MP 配置 type-handlers-package 扫描）。
二、完整代码逐行解析
java
运行
// 1. 类定义与继承
@Component // 交给Spring容器管理，可选（也可通过MP配置扫描）
public class StatusEnumTypeHandler extends BaseTypeHandler<StatusEnum> {
    // 泛型指定为目标枚举类型：StatusEnum，限定处理器只处理该枚举
方法 1：getNullableResult (ResultSet rs, String columnName)
作用：从 ResultSet（查询结果集）中，根据「字段名」读取数据库值，转换为 StatusEnum 枚举。
```java
运行
    // 从数据库读取值转换为枚举（根据字段名获取值）
    @Override
    public StatusEnum getNullableResult(ResultSet rs, String columnName) throws SQLException {
        // 步骤1：通过字段名从结果集获取数据库存储的字符串（如"启用"）
        String desc = rs.getString(columnName);
        // 步骤2：将数据库字符串转换为枚举
        return Arrays.stream(StatusEnum.values()) // 遍历枚举所有枚举值（DISABLE/ENABLE）
                .filter(e -> e.getDesc().equals(desc)) // 过滤：枚举的desc与数据库值匹配
                .findFirst() // 取第一个匹配的枚举（保证desc唯一）
                .orElse(null); // 无匹配时返回null（避免NoSuchElementException）
    }
关键参数 / 方法说明：
ResultSet rs：MyBatis 查询数据库后返回的结果集，包含所有查询字段的值；
String columnName：要转换的数据库字段名（如实体类中 status 对应的数据库字段 status_desc）；
rs.getString(columnName)：从结果集中按字段名读取字符串值（数据库存储的是 "启用"/"禁用"）；
Arrays.stream(StatusEnum.values())：将枚举的所有值（values() 返回数组）转为流，方便遍历过滤；
filter(...)：核心匹配逻辑，通过枚举的 getDesc() 方法对比数据库值；
findFirst().orElse(null)：安全获取匹配结果，避免无匹配时抛出异常。
方法 2：getNullableResult (ResultSet rs, int columnIndex)
作用：从 ResultSet 中，根据「字段下标」读取数据库值，转换为 StatusEnum 枚举（重载方法，适配不同取值方式）。
java
运行
    @Override
    public StatusEnum getNullableResult(ResultSet rs, int columnIndex) throws SQLException {
        // 步骤1：通过字段下标从结果集获取值（如下标3对应status_desc字段）
        String desc = rs.getString(columnIndex);
        // 步骤2：与方法1逻辑完全一致，转换为枚举
        return Arrays.stream(StatusEnum.values())
                .filter(e -> e.getDesc().equals(desc))
                .findFirst()
                .orElse(null);
    }
核心差异：
int columnIndex：数据库字段在查询结果中的下标（从 1 开始），而非字段名；
适用场景：MyBatis 底层可能通过下标取值（如手写 SQL 未指定别名时），需适配此场景。
方法 3：getNullableResult (CallableStatement cs, int columnIndex)
作用：从 CallableStatement（存储过程调用结果）中读取值，转换为枚举（适配存储过程场景）。
java
运行
    @Override
    public StatusEnum getNullableResult(CallableStatement cs, int columnIndex) throws SQLException {
        // 步骤1：从存储过程结果中按下标读取字符串值
        String desc = cs.getString(columnIndex);
        // 步骤2：同样的枚举转换逻辑
        return Arrays.stream(StatusEnum.values())
                .filter(e -> e.getDesc().equals(desc))
                .findFirst()
                .orElse(null);
    }
关键参数说明：
CallableStatement cs：调用数据库存储过程时的语句对象，用于获取存储过程的输出参数 / 返回值；
适用场景：项目中使用数据库存储过程查询数据时，MyBatis 会调用此方法转换枚举字段。
方法 4：setNonNullParameter (PreparedStatement ps, int i, StatusEnum parameter, JdbcType jdbcType)
作用：将 Java 枚举对象转换为数据库存储的字符串值，写入 PreparedStatement（用于新增 / 更新操作）。
java
运行
    // 从枚举转换为数据库存储值（写入数据库时调用）
    @Override
    public void setNonNullParameter(PreparedStatement ps, int i, StatusEnum parameter, JdbcType jdbcType) throws SQLException {
        // 步骤1：将枚举转换为数据库存储的字符串（如StatusEnum.ENABLE → "启用"）
        // 步骤2：将字符串设置到PreparedStatement的第i个参数位置
        ps.setString(i, parameter.getDesc());
    }
关键参数 / 方法说明：
PreparedStatement ps：MyBatis 执行新增 / 更新时的 SQL 预编译对象，用于设置参数；
int i：参数在 SQL 中的下标（从 1 开始，如 INSERT INTO user(status) VALUES(?) 中 ? 的位置）；
StatusEnum parameter：要写入数据库的枚举对象（如 StatusEnum.ENABLE）；
JdbcType jdbcType：数据库字段的 JDBC 类型（如 VARCHAR），此处未使用，仅为接口要求；
ps.setString(i, parameter.getDesc())：将枚举的描述字符串设置到 SQL 参数中，最终写入数据库。
三、核心设计逻辑与注意事项
1. 双向转换的完整性
该处理器覆盖了「读（查询）」和「写（新增 / 更新）」两个方向：
读数据库：3 个 getNullableResult 方法适配不同查询场景（字段名 / 下标、普通查询 / 存储过程）；
写数据库：setNonNullParameter 方法负责枚举 → 数据库字符串的转换。
2. 空值处理
方法名中的 Nullable 表示允许处理 null 值：
读取时：若数据库字段为 null，rs.getString(...) 返回 null，最终方法返回 null；
写入时：setNonNullParameter 仅处理非 null 枚举（若枚举为 null，MyBatis 会调用 setNull 方法，无需重写）。
3. 性能与健壮性
枚举遍历优化：Arrays.stream(StatusEnum.values()) 是流式遍历，枚举值数量少（通常几个），性能无损耗；
避免异常：使用 findFirst().orElse(null) 而非 findFirst().get()，防止无匹配值时抛出 NoSuchElementException；
唯一性保证：需确保枚举的 desc 字段唯一（如 "启用" 只能对应 StatusEnum.ENABLE），否则可能返回错误的枚举值。
4. 与 MP 内置处理器的区别
处理器类型	数据库存储值	核心依赖	适用场景
MybatisEnumTypeHandler	枚举的 @EnumValue 值	@EnumValue 注解	存储数字编码（如 0/1）
自定义 StatusEnumTypeHandler	枚举描述字符串	自定义匹配逻辑	存储中文 / 英文描述（如 "启用"）
5. 使用时的关键配置
方式 1：实体类字段指定处理器
java
运行
@Data
@TableName("user")
public class User {
    private Long id;
    private String username;
    // 指定自定义处理器，绑定枚举字段
    @TableField(typeHandler = StatusEnumTypeHandler.class)
    private StatusEnum status;
}
方式 2：全局扫描处理器（推荐，避免重复指定）
yaml
mybatis-plus:
  configuration:
    # 扫描自定义处理器所在包，MP自动加载
    type-handlers-package: com.example.handler
四、扩展优化建议
1. 缓存枚举映射（性能优化）
若枚举值较多，可在处理器初始化时缓存「描述字符串 → 枚举」的映射，避免每次查询都遍历枚举：
java
运行
@Component
public class StatusEnumTypeHandler extends BaseTypeHandler<StatusEnum> {
    // 缓存：描述字符串 → 枚举
    private final Map<String, StatusEnum> descToEnumMap;

    // 构造方法初始化缓存
    public StatusEnumTypeHandler() {
        this.descToEnumMap = Arrays.stream(StatusEnum.values())
                .collect(Collectors.toMap(StatusEnum::getDesc, e -> e));
    }

    // 读取时直接从缓存获取
    @Override
    public StatusEnum getNullableResult(ResultSet rs, String columnName) throws SQLException {
        String desc = rs.getString(columnName);
        return descToEnumMap.get(desc); // 无需遍历，直接获取
    }
    // 其他get方法同理，改用缓存获取...
}
2. 自定义异常（增强可读性）
若希望无匹配值时抛出明确的业务异常，而非返回 null：
java
运行
return descToEnumMap.getOrDefault(desc, () -> {
    throw new BusinessException("无效的状态描述：" + desc);
});
五、总结
该自定义枚举处理器的核心是实现 BaseTypeHandler 的 4 个方法，完成「数据库字符串 ↔ Java 枚举」的双向转换：
读数据库：3 个 getNullableResult 方法适配不同查询场景，核心是「数据库值 → 枚举」的匹配；
写数据库：setNonNullParameter 方法实现「枚举 → 数据库值」的转换；
设计上兼顾了兼容性（适配不同查询方式）、健壮性（避免空指针 / 无匹配异常）和易用性（Spring 管理）。
MP封页插件功能企业级代码演示以及解释
MyBatis-Plus 分页插件 企业级使用教程（代码 + 详解）
MyBatis-Plus（MP）的分页插件是企业开发中高频使用的核心功能，相比原生 MyBatis 分页，MP 分页插件无需手写 LIMIT 语句，支持单表分页、多表联查分页、自定义 SQL 分页，且适配主流数据库（MySQL/Oracle/PostgreSQL 等）。以下是企业级完整代码演示及核心逻辑解释。
一、核心依赖（基础前提）
确保项目已引入 MP 核心依赖（分页插件内置在 mybatis-plus-boot-starter 中，无需额外依赖）：
xml
<dependency>
    <groupId>com.baomidou</groupId>
    <artifactId>mybatis-plus-boot-starter</artifactId>
    <version>3.5.5</version> <!-- 企业级推荐稳定版 -->
</dependency>
二、企业级分页插件配置（核心）
1. 分页插件配置类（适配多数据库 + 性能优化）
企业开发中需考虑「多数据库适配、分页合理化、性能控制」，以下是标准化配置：
java
运行
import com.baomidou.mybatisplus.annotation.DbType;
import com.baomidou.mybatisplus.extension.plugins.MybatisPlusInterceptor;
import com.baomidou.mybatisplus.extension.plugins.inner.PaginationInnerInterceptor;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

/**
 * MP 全局配置类（企业级分页插件核心配置）
 * 说明：MyBatis-Plus 3.4.0+ 推荐使用 MybatisPlusInterceptor 替代旧版 PaginationInterceptor
 */
@Configuration
public class MyBatisPlusConfig {

    /**
     * 分页插件核心 Bean
     */
    @Bean
    public MybatisPlusInterceptor mybatisPlusInterceptor() {
        MybatisPlusInterceptor interceptor = new MybatisPlusInterceptor();
        
        // 1. 创建分页拦截器，指定数据库类型（关键：适配不同数据库的分页语法）
        PaginationInnerInterceptor paginationInterceptor = new PaginationInnerInterceptor();
        
        // ========== 企业级核心配置项 ==========
        // （1）指定数据库类型（必填，否则分页语法可能错误）
        paginationInterceptor.setDbType(DbType.MYSQL); 
        // 可选值：DbType.MYSQL / DbType.ORACLE / DbType.POSTGRE_SQL / DbType.SQL_SERVER
        
        // （2）溢出分页处理：默认false（超出总页数返回空页），企业级推荐设置为true（超出后返回最后一页）
        paginationInterceptor.setOverflow(true);
        
        // （3）单页最大条数限制（防止恶意请求，如一页查10万条）
        paginationInterceptor.setMaxLimit(1000L); // 单页最多查1000条
        
        // （4）计数优化：默认true（自动优化 COUNT 查询，如单表用 COUNT(1) 替代 COUNT(*)）
        paginationInterceptor.setOptimizeJoin(true); // 多表联查时优化 COUNT 语句
        
        // 2. 将分页拦截器添加到 MP 拦截器链
        interceptor.addInnerInterceptor(paginationInterceptor);
        
        return interceptor;
    }
}
配置项解释（企业级关注点）
配置项	作用 & 企业级意义
setDbType(DbType)	适配不同数据库分页语法（MySQL 用 LIMIT，Oracle 用 ROWNUM，PostgreSQL 用 LIMIT/OFFSET）
setOverflow(true)	防止「页码超出总页数」返回空数据（如总共有 5 页，请求第 10 页时返回第 5 页数据，提升用户体验）
setMaxLimit(1000L)	限制单页最大条数，防止恶意请求（如pageSize=100000）导致数据库压力过大
setOptimizeJoin(true)	优化多表联查的 COUNT 语句（避免 COUNT (*) 全表扫描，改为 COUNT (主键)）
三、企业级分页使用场景（完整代码）
场景 1：单表分页（最常用，Service 层封装）
步骤 1：定义分页查询 DTO（企业级规范，统一入参）
java
运行
import lombok.Data;

/**
 * 用户分页查询入参 DTO（企业级：入参和出参分离，避免直接用实体类）
 */
@Data
public class UserPageQueryDTO {
    private Integer pageNum; // 当前页码（默认1）
    private Integer pageSize; // 每页条数（默认10）
    private String username; // 用户名模糊查询（可选）
    private Integer age; // 年龄精确查询（可选）
    private Integer minAge; // 年龄最小值（可选）
}
步骤 2：Service 层实现分页逻辑（企业级封装）
java
运行
import com.baomidou.mybatisplus.core.conditions.query.LambdaQueryWrapper;
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import com.baomidou.mybatisplus.extension.service.impl.ServiceImpl;
import org.springframework.stereotype.Service;
import org.springframework.util.StringUtils;

@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    /**
     * 企业级单表分页查询
     * @param queryDTO 分页查询条件
     * @return 分页结果（包含数据+分页元信息）
     */
    @Override
    public IPage<User> pageQuery(UserPageQueryDTO queryDTO) {
        // ========== 1. 分页参数处理（企业级：默认值+合法性校验） ==========
        int pageNum = queryDTO.getPageNum() == null ? 1 : queryDTO.getPageNum();
        int pageSize = queryDTO.getPageSize() == null ? 10 : queryDTO.getPageSize();
        // 合法性校验：页码≥1，每页条数≤1000（匹配插件的maxLimit）
        pageNum = Math.max(pageNum, 1);
        pageSize = Math.min(pageSize, 1000);
        
        // ========== 2. 创建分页对象（核心：Page<T> 封装分页元信息） ==========
        // Page<实体类>：第一个参数=当前页，第二个参数=每页条数
        Page<User> page = new Page<>(pageNum, pageSize);
        
        // ========== 3. 构建查询条件（企业级：动态条件，避免硬编码） ==========
        LambdaQueryWrapper<User> wrapper = new LambdaQueryWrapper<User>()
                // 用户名模糊查询（非空才拼接条件）
                .like(StringUtils.hasText(queryDTO.getUsername()), User::getUsername, queryDTO.getUsername())
                // 年龄精确查询（非空才拼接）
                .eq(queryDTO.getAge() != null, User::getAge, queryDTO.getAge())
                // 年龄范围查询（最小值非空）
                .ge(queryDTO.getMinAge() != null, User::getAge, queryDTO.getMinAge())
                // 排序（企业级：默认按ID降序）
                .orderByDesc(User::getId);
        
        // ========== 4. 执行分页查询（核心：MP自动拼接分页SQL） ==========
        // 方式1：IService 内置分页方法（推荐，简化代码）
        IPage<User> userPage = this.page(page, wrapper);
        
        // 方式2：直接调用Mapper的selectPage方法（底层和page()一致）
        // IPage<User> userPage = baseMapper.selectPage(page, wrapper);
        
        return userPage;
    }
}
步骤 3：Controller 层返回分页结果（企业级出参）
java
运行
import com.baomidou.mybatisplus.core.metadata.IPage;
import org.springframework.web.bind.annotation.PostMapping;
import org.springframework.web.bind.annotation.RequestBody;
import org.springframework.web.bind.annotation.RestController;

/**
 * 企业级分页接口规范：返回分页元信息+数据列表
 */
@RestController
public class UserController {

    @Autowired
    private UserService userService;

    @PostMapping("/user/page")
    public Result<IPage<User>> pageQuery(@RequestBody UserPageQueryDTO queryDTO) {
        IPage<User> userPage = userService.pageQuery(queryDTO);
        // 企业级统一返回结果（Result为自定义全局返回类）
        return Result.success(userPage);
    }
}
步骤 4：自定义全局返回类（企业级规范）
java
运行
import lombok.Data;

/**
 * 全局统一返回结果
 * 企业级：包含状态码、提示信息、分页数据
 */
@Data
public class Result<T> {
    private Integer code; // 状态码（200=成功，500=失败）
    private String msg; // 提示信息
    private T data; // 数据（分页结果/列表/单个对象）

    // 成功返回
    public static <T> Result<T> success(T data) {
        Result<T> result = new Result<>();
        result.setCode(200);
        result.setMsg("操作成功");
        result.setData(data);
        return result;
    }

    // 失败返回
    public static <T> Result<T> fail(String msg) {
        Result<T> result = new Result<>();
        result.setCode(500);
        result.setMsg(msg);
        result.setData(null);
        return result;
    }
}
场景 2：多表联查 / 自定义 SQL 分页（企业级高频场景）
单表分页满足不了复杂业务（如用户 + 部门联查），需自定义 SQL 结合分页插件使用。
步骤 1：Mapper 层定义自定义分页方法
java
运行
import com.baomidou.mybatisplus.core.metadata.IPage;
import com.baomidou.mybatisplus.extension.plugins.pagination.Page;
import org.apache.ibatis.annotations.Param;
import org.apache.ibatis.annotations.Select;

public interface UserMapper extends BaseMapper<User> {

    /**
     * 自定义SQL分页（用户+部门联查）
     * 企业级注意：
     * 1. 参数必须包含 Page<T> 对象（MP会自动解析分页参数）
     * 2. @Param 注解绑定参数，方便XML中引用
     * 3. 返回值必须是 IPage<T> 或 Page<T>
     */
    @Select("SELECT u.id, u.username, u.age, d.dept_name " +
            "FROM user u LEFT JOIN dept d ON u.dept_id = d.id " +
            "WHERE u.is_deleted = 0 " +
            "AND (u.username LIKE CONCAT('%', #{username}, '%') OR #{username} IS NULL) " +
            "ORDER BY u.id DESC")
    IPage<UserDeptVO> selectUserDeptPage(
            Page<UserDeptVO> page, // 分页对象（MP自动拼接LIMIT）
            @Param("username") String username // 查询条件
    );

    // 也可通过XML编写复杂SQL（推荐：复杂SQL分离，便于维护）
    // IPage<UserDeptVO> selectUserDeptPage(Page<UserDeptVO> page, @Param("username") String username);
}
步骤 2：定义 VO（视图对象，企业级：联查结果封装）
java
运行
import lombok.Data;

/**
 * 用户+部门联查VO（企业级：避免返回多余字段，只返回前端需要的）
 */
@Data
public class UserDeptVO {
    private Long id;
    private String username;
    private Integer age;
    private String deptName; // 部门名称
}
步骤 3：Service 层实现自定义分页
java
运行
@Service
public class UserServiceImpl extends ServiceImpl<UserMapper, User> implements UserService {

    /**
     * 企业级多表联查分页
     */
    @Override
    public IPage<UserDeptVO> pageUserDept(UserPageQueryDTO queryDTO) {
        // 1. 分页参数处理（同单表）
        int pageNum = Math.max(queryDTO.getPageNum() == null ? 1 : queryDTO.getPageNum(), 1);
        int pageSize = Math.min(queryDTO.getPageSize() == null ? 10 : queryDTO.getPageSize(), 1000);
        Page<UserDeptVO> page = new Page<>(pageNum, pageSize);
        
        // 2. 执行自定义SQL分页（MP自动为自定义SQL拼接LIMIT）
        IPage<UserDeptVO> userDeptPage = baseMapper.selectUserDeptPage(page, queryDTO.getUsername());
        
        return userDeptPage;
    }
}
步骤 4：XML 版自定义 SQL（复杂 SQL 推荐，企业级规范）
若 SQL 逻辑复杂（如多表联查、子查询），建议写在 XML 中：
xml
<!-- resources/mapper/UserMapper.xml -->
<?xml version="1.0" encoding="UTF-8"?>
<!DOCTYPE mapper PUBLIC "-//mybatis.org//DTD Mapper 3.0//EN" "http://mybatis.org/dtd/mybatis-3-mapper.dtd">
<mapper namespace="com.example.mapper.UserMapper">

    <select id="selectUserDeptPage" resultType="com.example.vo.UserDeptVO">
        SELECT u.id, u.username, u.age, d.dept_name 
        FROM user u LEFT JOIN dept d ON u.dept_id = d.id 
        WHERE u.is_deleted = 0 
        <if test="username != null and username != ''">
            AND u.username LIKE CONCAT('%', #{username}, '%')
        </if>
        ORDER BY u.id DESC
    </select>

</mapper>
四、分页结果解析（企业级必懂）
调用分页接口后，返回的 IPage<T> 包含以下核心属性（前端分页控件需用到）：
json
{
  "code": 200,
  "msg": "操作成功",
  "data": {
    "records": [ // 当前页数据列表
      {"id": 1, "username": "张三", "age": 25, "deptName": "研发部"},
      {"id": 2, "username": "李四", "age": 30, "deptName": "产品部"}
    ],
    "total": 100, // 总记录数（企业级：用于计算总页数）
    "size": 10,   // 每页条数
    "current": 1, // 当前页码
    "pages": 10,  // 总页数（total/size 向上取整）
    "hasNext": true, // 是否有下一页（企业级：前端分页控件用）
    "hasPrevious": false // 是否有上一页
  }
}
IPage<T> 核心方法 / 属性说明
属性 / 方法	作用
records	当前页的数据列表（核心，前端展示数据）
total	符合条件的总记录数（非当前页，企业级：用于分页控件的「共 XX 条」）
size	每页条数（前端传入的 pageSize）
current	当前页码（前端传入的 pageNum）
pages	总页数（自动计算：total /size 向上取整）
hasNext()	是否有下一页（current < pages）
hasPrevious()	是否有上一页（current > 1）
getOptimizeCountSql()	是否优化 COUNT SQL（企业级：性能相关）
五、企业级优化与注意事项
1. 性能优化（核心）
避免 COUNT (*) 全表扫描：MP 分页插件默认优化为 COUNT(1)，多表联查时建议手动指定 COUNT(u.id)（u 为主表别名）；
分页查询加索引：对查询条件字段（如 username、age）、排序字段（如 id）建立索引，避免分页时全表扫描；
大数据量分页优化：若总记录数超 100 万，避免用 SELECT COUNT(*) 计算总数（可缓存总数 / 异步更新总数），或用「游标分页」替代传统分页；
限制分页参数：通过 setMaxLimit(1000) + 代码层校验，防止 pageSize 过大导致数据库压力。
2. 兼容性处理
Oracle 分页适配：Oracle 分页语法为 ROWNUM，需确保 setDbType(DbType.ORACLE)，MP 会自动拼接 WHERE ROWNUM <= ? AND ROWNUM >= ?；
分页参数默认值：代码层必须给 pageNum/pageSize 设置默认值（如 pageNum=1，pageSize=10），避免空指针；
溢出处理：setOverflow(true) 确保页码超出总页数时返回最后一页，而非空数据（提升用户体验）。
3. 企业级规范
入参用 DTO：分页查询条件封装为 DTO，避免直接用实体类（防止字段冗余 / 安全问题）；
出参用 VO：返回前端的分页结果封装为 VO，只返回需要的字段（减少数据传输）；
统一返回结果：所有分页接口返回全局 Result<IPage<T>>，前端无需适配不同格式；
异常处理：分页查询时捕获异常（如 SQL 错误），返回友好提示（如 Result.fail("分页查询失败：" + e.getMessage())）。
六、扩展：分页插件高级用法
1. 手动分页（无查询条件，仅分页）
java
运行
// 仅分页，无查询条件
Page<User> page = new Page<>(1, 10);
IPage<User> userPage = baseMapper.selectPage(page, null);
2. 分页查询后处理数据（企业级：数据脱敏 / 格式转换）
java
运行
IPage<User> userPage = this.page(page, wrapper);
// 对当前页数据进行脱敏（如手机号隐藏中间4位）
userPage.getRecords().forEach(user -> {
    String phone = user.getPhone();
    if (StringUtils.hasText(phone)) {
        user.setPhone(phone.replaceAll("(\\d{3})\\d{4}(\\d{4})", "$1****$2"));
    }
});
3. 自定义 COUNT SQL（解决多表联查 COUNT 性能问题）
java
运行
// 自定义 COUNT SQL（默认 COUNT(*) 性能差时使用）
page.setCountSql("SELECT COUNT(DISTINCT u.id) FROM user u LEFT JOIN dept d ON u.dept_id = d.id WHERE u.is_deleted = 0");
IPage<UserDeptVO> userDeptPage = baseMapper.selectUserDeptPage(page, username);
总结
MP 分页插件的企业级使用核心是：
标准化配置：适配数据库、限制单页条数、处理溢出分页；
规范入参出参：用 DTO 接收条件，VO 封装结果，统一返回格式；
性能优先：优化 COUNT SQL、加索引、限制分页参数；
兼容复杂场景：支持单表 / 多表 / 自定义 SQL 分页，适配不同数据库。
以上代码可直接用于生产环境，符合企业级开发规范（可读性、可维护性、性能、兼容性均考虑到位）。
```