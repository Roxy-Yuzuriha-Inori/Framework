# 起步依赖
Springboot准备了几套起步依赖，然后可以通过maven依赖传递获取一整套的依赖

# 自动配置
Springboot自动配置了几个常见的依赖，可以直接注入，不用@Bean作用于方法声明第三方依赖
## 方案一
组件扫描的bean的范围是启动类所在的包及其自包<br>
而现在我导入了另外一个包，并且包中的类已用@Componet注解标记成为bean<br>
此时默认组件扫描bean的范围扫不到该包，不能直接@Autowired去注入引入包的bean<br>
所以要在启动类上添加@ComponetScan(basePackages={"导入进来的包名","启动类包名"})实现自动配置

## 方案二@Import
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

## 方案三@Enable...
- 作用于启动类
- 编写包的人才知道要自动配置那些类，所以@Enable...封装了@import注解
```java
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Import(MyImportSelector.class)//指定要导入哪些bean对象或配置类
public @interface EnableHeaderConfig { 
}
```




# 配置优先级
