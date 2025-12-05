# AOP核心概念
## 连接点
可被AOP控制的方法的位置
## 通知
要对连接点施加的重复逻辑
## 切入点
要放通知的位置，通过切入点表达式实现
## 切面
切入点+通知
## 目标对象
通知应用的切入点的对象

# AOP底层机制
将目标对象变成一个代理对象，该代理对象增加了AOP的通知方法，以后注入的其实是代理对象，调用的是代理对象的方法

# AOP进阶
## 通知类型
### @Around
环绕通知，此注解标注的通知方法在目标方法前、后都被执行
### @Before
前置通知，此注解标注的通知方法在目标方法前被执行
### @After
后置通知，此注解标注的通知方法在目标方法后被执行，无论是否有异常都会执行
### @AfterReturning
返回后通知，此注解标注的通知方法在目标方法后被执行，有异常不会执行
### @AfterThrowing
异常后通知，此注解标注的通知方法发生异常后执行

```java
@Slf4j
@Component
@Aspect
public class MyAspect1 {
    //前置通知
    @Before("execution(* com.itheima.service.*.*(..))")
    public void before(JoinPoint joinPoint){
        log.info("before ...");

    }

    //环绕通知
    @Around("execution(* com.itheima.service.*.*(..))")
    public Object around(ProceedingJoinPoint proceedingJoinPoint) throws Throwable {
        log.info("around before ...");

        //调用目标对象的原始方法执行,接收原始方法的返回值
        Object result = proceedingJoinPoint.proceed();
        
        //原始方法如果执行时有异常，环绕通知中的后置代码不会在执行了
        
        log.info("around after ...");
        return result;
    }

    //后置通知
    @After("execution(* com.itheima.service.*.*(..))")
    public void after(JoinPoint joinPoint){
        log.info("after ...");
    }

    //返回后通知（程序在正常执行的情况下，会执行的后置通知）
    @AfterReturning("execution(* com.itheima.service.*.*(..))")
    public void afterReturning(JoinPoint joinPoint){
        log.info("afterReturning ...");
    }

    //异常通知（程序在出现异常的情况下，执行的后置通知）
    @AfterThrowing("execution(* com.itheima.service.*.*(..))")
    public void afterThrowing(JoinPoint joinPoint){
        log.info("afterThrowing ...");
    }
}

```
*** 无异常执行顺序 ***<br>
around before<br>
before<br>
afterReturning<br>
after<br>
around after<br>

*** 异常执行顺序 ***<br>
around before<br>
before<br>
afterThrowing<br>
after<br>

## 通知顺序
  - 目标方法前的通知方法：字母排名靠前的先执行
  - 目标方法后的通知方法：字母排名靠前的后执行
### @Order(2)
- 作用于切面类
- 前置通知：数字越小先执行; 后置通知：数字越小越后执行  

## 切入点表达式
### @PointCut
- 作用于方法
- 抽取公共切入点表达式
- 通过"方法名（）"引用  如@Before("pt()")
```java
@Slf4j
@Component
@Aspect
public class MyAspect1 {

    //切入点方法（公共的切入点表达式）
    @Pointcut("execution(* com.itheima.service.*.*(..))")
    private void pt(){}

    //前置通知（引用切入点）
    @Before("pt()")
    public void before(JoinPoint joinPoint){
        log.info("before ...");

    }
}
```
*** 当外部其他切面类中也要引用当前类中的切入点表达式，就需要把private改为public ***
```java
@Slf4j
@Component
@Aspect
public class MyAspect2 {
    //引用MyAspect1切面类中的切入点表达式,切面1的表达式要改为public
    @Before("com.itheima.aspect.MyAspect1.pt()")
    public void before(){
        log.info("MyAspect2 -> before ...");
    }
}
```
### execution
*** 带?的表示可以省略的部分 ***<br>
- 访问修饰符：可省略（比如: public、protected）
- 包名.类名： 可省略
- throws 异常：可省略（注意是方法上声明抛出的异常，不是实际抛出的异常）

*** 可以使用通配符描述切入点 ***
- * ：单个独立的任意符号，可以通配任意返回值、包名、类名、方法名、任意类型的一个参数，也可以通配包、类、方法名的一部分
- .. ：多个连续的任意符号，可以通配任意层级的包，或任意类型、任意个数的参数

切入点表达式的语法规则：
1. 方法的访问修饰符可以省略
2. 返回值可以使用*号代替（任意返回值类型）
3. 包名可以使用*号代替，代表任意包（一层包使用一个*）
4. 使用..配置包名，标识此包以及此包下的所有子包
5. 类名可以使用*号代替，标识任意类
6. 方法名可以使用*号代替，表示任意方法
7. 可以使用 *  配置参数，一个任意类型的参数
8. 可以使用.. 配置参数，任意个任意类型的参数
```java
execution(访问修饰符?  返回值  包名.类名.?方法名(方法参数) throws 异常?)
```
```java
//1.省略包名
execution(* com..DeptServiceImpl.delete(java.lang.Integer))  

//2.省略类名
execution(* com..*.delete(java.lang.Integer))

//3.*可以表示方法名的一部分
execution(* com.itheima.service.impl.DeptServiceImpl.find*(..))

```
### @annotation
- 代替切点表达式，作用于方法
- 将自定义注解加在要加强的方法上

```java

//自定义注解加在方法上
@Slf4j
@Service
public class DeptServiceImpl implements DeptService {
    @Autowired
    private DeptMapper deptMapper;

    @Override
    @LogOperation //自定义注解（表示：当前方法属于目标方法）
    public List<Dept> list() {
        List<Dept> deptList = deptMapper.list();
        return deptList;
    }
}

//代替切点表达式，作用于方法
@Slf4j
@Component
@Aspect
public class MyAspect6 {
    //针对list方法、delete方法进行前置通知和后置通知

    //前置通知   copy reference复制全类名
    @Before("@annotation(com.itheima.anno.LogOperation)")
    public void before(){
        log.info("MyAspect6 -> before ...");
    }
    
    //后置通知
    @After("@annotation(com.itheima.anno.LogOperation)")
    public void after(){
        log.info("MyAspect6 -> after ...");
    }
}

```
#### 自定义注解
- 放在anno包下
- @Target是原注解，修饰注解的注解。后面参数代表该注解作用于方法上
- Retention也是原注解，代表这个注解什么时候生效。后面参数代表是在运行是生效

```java
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogOperation{
}
```
## 连接点
- 对于@Around通知，获取连接点信息只能使用ProceedingJoinPoint类型
- 对于其他四种通知，获取连接点信息只能使用JoinPoint，它是ProceedingJoinPoint的父类型
```java

@Around("@Annotation(log)")
public object around(ProceedingJoinPoint joinPoint,LogOperation log){
        OperateLog operateLog = new OperateLog();
        //获取目标对象的全类名  如com.itheima.service.UserService
        operateLog.setClassName(joinPoint.getTarget().getClass().getName());
        //获取方法签名的方法名 
        operateLog.setMethodName(joinPoint.getSignature().getName());
        //获取目标方法运行参数
        operateLog.setMethodParams(Arrays.toString(joinPoint.getArgs()));
}
```


# AOP案例
```xml
<!-- 引入AOP依赖 -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-aop</artifactId>
</dependency>

```
```java
//根据数据库表建立实体类
package com.itheima.pojo;

import lombok.AllArgsConstructor;
import lombok.Data;
import lombok.NoArgsConstructor;
import java.time.LocalDateTime;

@Data
@NoArgsConstructor
@AllArgsConstructor
public class OperateLog {
    private Integer id; //ID
    private Integer operateEmpId; //操作人ID
    private LocalDateTime operateTime; //操作时间
    private String className; //操作类名
    private String methodName; //操作方法名
    private String methodParams; //操作方法参数
    private String returnValue; //操作方法返回值
    private Long costTime; //操作耗时
}

//准备Mapper接口
package com.itheima.mapper;

import com.itheima.pojo.OperateLog;
import org.apache.ibatis.annotations.Insert;
import org.apache.ibatis.annotations.Mapper;

@Mapper
public interface OperateLogMapper {
    
    //插入日志数据
    @Insert("insert into operate_log (operate_emp_id, operate_time, class_name, method_name, method_params, return_value, cost_time) " +
            "values (#{operateEmpId}, #{operateTime}, #{className}, #{methodName}, #{methodParams}, #{returnValue}, #{costTime});")
    public void insert(OperateLog log);
    
}

//自定义AOP注解
/**
 *  自定义注解，用于标识哪些方法需要记录日志
 */
@Target(ElementType.METHOD)
@Retention(RetentionPolicy.RUNTIME)
public @interface LogOperation {
}

//切面类
import com.itheima.anno.LogOperation;
import com.itheima.mapper.OperateLogMapper;
import com.itheima.pojo.OperateLog;
import org.aspectj.lang.ProceedingJoinPoint;
import org.aspectj.lang.annotation.Around;
import org.aspectj.lang.annotation.Aspect;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.stereotype.Component;
import java.time.LocalDateTime;
import java.util.Arrays;

@Aspect
@Component
public class OperationLogAspect {

    @Autowired
    private OperateLogMapper operateLogMapper;

    // 环绕通知
    @Around("@annotation(log)")
    //形参LogOperation log是日志实体，可以调用log.value(); // 获取注解中的描述信息
    public Object around(ProceedingJoinPoint joinPoint, LogOperation log) throws Throwable {
        // 记录开始时间
        long startTime = System.currentTimeMillis();
        // 执行方法
        Object result = joinPoint.proceed();
        // 当前时间
        long endTime = System.currentTimeMillis();
        // 耗时
        long costTime = endTime - startTime;

        // 构建日志对象
        OperateLog operateLog = new OperateLog();
        operateLog.setOperateEmpId(getCurrentUserId()); // 需要实现 getCurrentUserId 方法
        operateLog.setOperateTime(LocalDateTime.now());
        operateLog.setClassName(joinPoint.getTarget().getClass().getName());
        operateLog.setMethodName(joinPoint.getSignature().getName());
        operateLog.setMethodParams(Arrays.toString(joinPoint.getArgs()));
        operateLog.setReturnValue(result.toString());
        operateLog.setCostTime(costTime);

        // 插入日志
        operateLogMapper.insert(operateLog);
        return result;
    }
    
    // 示例方法，从ThreadLocal获取当前用户ID
    private int getCurrentUserId() {
          return CurrentHolder.getCurrentId();
   
    }
}

//ThreadLocalUtils
package com.itheima.utils;

public class CurrentHolder {

    private static final ThreadLocal<Integer> CURRENT_LOCAL = new ThreadLocal<>();
    //存
    public static void setCurrentId(Integer employeeId) {
        CURRENT_LOCAL.set(employeeId);
    }
    //取
    public static Integer getCurrentId() {
        return CURRENT_LOCAL.get();
    }
    //用完释放
    public static void remove() {
        CURRENT_LOCAL.remove();
    }
}
//统一拦截器将共享数据存入ThreadLocal

@Slf4j
@WebFilter(urlPatterns = "/*")
public class TokenFilter implements Filter {
    @Override
    public void doFilter(ServletRequest servletRequest, ServletResponse servletResponse, FilterChain filterChain) throws IOException, ServletException {
        HttpServletRequest request = (HttpServletRequest) servletRequest;
        HttpServletResponse response = (HttpServletResponse) servletResponse;

        //1. 获取请求的url地址
        String uri = request.getRequestURI(); // /employee/login
        //String url = request.getRequestURL().toString(); // http://localhost:8080/employee/login

        //2. 判断是否是登录请求, 如果url地址中包含 login, 则说明是登录请求, 放行
        if (uri.contains("login")) {
            log.info("登录请求, 放行");
            filterChain.doFilter(request, response);
            return;
        }

        //3. 获取请求中的token
        String token = request.getHeader("token");

        //4. 判断token是否为空, 如果为空, 响应401状态码
        if (token == null || token.isEmpty()) {
            log.info("token为空, 响应401状态码");
            response.setStatus(401); // 响应401状态码
            return;
        }

        //5. 如果token不为空, 调用JWtUtils工具类的方法解析token, 如果解析失败, 响应401状态码
        try {
            //从JWT解析数据存入ThreadLocal
            Claims claims = JwtUtils.parseJWT(token);
            Integer empId = Integer.valueOf(claims.get("id").toString());
            CurrentHolder.setCurrentId(empId);
            log.info("token解析成功, 放行");
        } catch (Exception e) {
            log.info("token解析失败, 响应401状态码");
            response.setStatus(401);
            return;
        }

        //6. 放行
        filterChain.doFilter(request, response);

        //7. 清空当前线程绑定的id
        CurrentHolder.remove();
    }
}
```
## ThreadLocal
- 一次请求对应一个线程，通过ThreadLocal进行数据共享
