# 配置优先级
优先级(从低到高)：<br>
- application.yaml（忽略）
- application.yml
- application.properties
- java系统属性（-Dxxx=xxx）
- 命令行参数（--xxx=xxx）

*** java系统属性和命令行参数 ***<br>
- 可在启动程序的配置信息Modify options - Add VM options & Program arguments进行配置
- 可cmd对jar包进行配置 java -Dserver.port=9000  -jar XXXXX.jar  --server.port=10010

# 起步依赖
Springboot准备了几套起步依赖，然后可以通过maven依赖传递获取一整套的依赖

# 自动配置
Springboot自动配置了几个常见的依赖，可以直接注入，不用@Bean作用于方法声明第三方依赖
### 方案一
组件扫描的bean的范围是启动类所在的包及其自包<br>
而现在我导入了另外一个包，并且包中的类已用@Componet注解标记成为bean<br>
此时默认组件扫描bean的范围扫不到该包，不能直接@Autowired去注入引入包的bean<br>
所以要在启动类上添加@ComponetScan(basePackages={"导入进来的包名","启动类包名"})实现自动配置

### 方案二@Import
- 作用在启动上
- @Import(TokenParser.class) 导入普通类完成将该类放到IOC容器中
- @Import(HeaderConfig.class)导入配置类完成将配置类中所有类放到IOC容器中
```java
//配置类
@Configuration
public class HeaderConfig {
    @Bean
    public HeaderParser headerParser(){
        return new HeaderParser();
    }

    @Bean
    public HeaderGenerator headerGenerator(){
        return new HeaderGenerator();
    }
}
```
- @Import(MyImportSelector.class)导入ImportSelector接口实现类,该类中可以批量放配置类
```java
public class MyImportSelector implements ImportSelector {
    public String[] selectImports(AnnotationMetadata importingClassMetadata) {
        //返回值字符串数组（数组中封装了全限定名称的类）
        return new String[]{"com.example.HeaderConfig"};
    }
}
```

### 方案三@Enable...
- 作用于启动类
- 编写包的人才知道要自动配置那些类，所以@Enable...封装了@import注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyImportSelector.class)//指定要导入哪些bean对象或配置类
public @interface EnableHeaderConfig { 
}
```
## 自动配置原理
### @SpringBootApplication
- 封装了三个注解
- @SpringBootConfiguration 封装@Configuration，启动类也是配置类
- @EnableAutoConfiguration 封装@import注解(Import注解中指定了一个ImportSelector接口的实现类),读取配置类
- @ComponentScan 组件扫描启动类所在包及其子包

### @Conditional
- 作用于类或方法
- 限定bean注入容器的条件
  - @ConditionalOnClass：判断环境中有对应字节码文件，才注册bean到IOC容器。
```java
  @Configuration
public class HeaderConfig {

    @Bean
    @ConditionalOnClass(name="io.jsonwebtoken.Jwts")//环境中存在指定的这个类，才会将该bean加入IOC容器
    public HeaderParser headerParser(){
        return new HeaderParser();
    }
    
    //省略其他代码...
}
```
  - @ConditionalOnMissingBean：判断环境中没有对应的bean(类型或名称)，才注册bean到IOC容器。
```java
@Configuration
public class HeaderConfig {
        
    @Bean
    @ConditionalOnMissingBean //不存在该类型的bean，才会将该bean加入IOC容器
    public HeaderParser headerParser(){
        return new HeaderParser();
    }
    
    //省略其他代码...
}
```
  - @ConditionalOnProperty：判断配置文件中有对应属性和值，才注册bean到IOC容器。
```java
@Configuration
public class HeaderConfig {

    @Bean
    //配置文件中存在指定属性名与值，才会将bean加入IOC容器
    @ConditionalOnProperty(name ="name",havingValue = "itheima")
    public HeaderParser headerParser(){
        return new HeaderParser();
    }

    @Bean
    public HeaderGenerator headerGenerator(){
        return new HeaderGenerator();
    }
}
```
# 自定义start
- 两个模块
- spring-boot-starter  或  xxx-spring-boot-starter 这个模块主要是依赖管理的功能，只需要一个pom文件，其他只需导入start这一个模块
- spring-boot-autoconfigure 或 xxxx-spring-boot-autoconfigure 主要是起到自动配置的作用，编写自动配置的核心代码

```xml
<!-- starter pom -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.8</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-starter</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
<!-- 导入autoconfigure模块 -->
        <dependency>
            <groupId>com.aliyun.oss</groupId>
            <artifactId>aliyun-oss-spring-boot-autoconfigure</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>

</project>
```
```xml
<!--autoconfigure pom  -->
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>3.2.8</version>
        <relativePath/> <!-- lookup parent from repository -->
    </parent>

    <groupId>com.aliyun.oss</groupId>
    <artifactId>aliyun-oss-spring-boot-autoconfigure</artifactId>
    <version>0.0.1-SNAPSHOT</version>

    <properties>
        <java.version>17</java.version>
    </properties>

    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter</artifactId>
        </dependency>
    </dependencies>
</project>
```
- 实现autoconfigure的包里有Properties 实体类，功能类和一个配置类
```java
//配置类
package com.aliyun.oss;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@EnableConfigurationProperties(AliyunOSSProperties.class)  //声明Properties 实体类为IOCbean
public class AliyunOSSAutoConfiguration {
    
    @Bean   //声明功能类为bean
    public AliyunOSSOperator aliyunOSSOperator(AliyunOSSProperties aliyunOSSProperties) {
        return new AliyunOSSOperator(aliyunOSSProperties);
    }
    
}

//Properties 实体类
@ConfigurationProperties(prefix="aaa.bbb")
public class AbcProperties(){
    private String a;
    private String b;
}
```
- 在autoconfigure 模块中的resources下，新建自动配置文件 META-INF/spring/org.springframework.boot.autoconfigure.AutoConfiguration.imports配置com.aliyun.oss.AliyunOSSAutoConfiguration
