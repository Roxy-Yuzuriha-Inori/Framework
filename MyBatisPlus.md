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