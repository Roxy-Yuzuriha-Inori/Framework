# 基础
- 作用：项目构建与依赖管理
```xml
<dependencies>
    <!-- 依赖 : spring-context -->
    <dependency>
       <!-- GroupID组织名 -->
        <groupId>org.springframework</groupId>
       <!-- ArtifactID依赖名 -->        
        <artifactId>spring-context</artifactId>
       <!-- Version主版本号.次版本号.修订号     主版本号（模块的改变）次版本号（功能改变）修订号（改bug）-->    
        <version>6.1.4</version>
        <!-- Scope  https://maven.apache.org/guides/introduction/introduction-to-dependency-mechanism.html#dependency-scope -->
        <scope></scope>
            <!--排除依赖, 主动断开依赖的资源-->
        <exclusions>
          <exclusion>
            <groupId>io.micrometer</groupId>
            <artifactId>micrometer-observation</artifactId>
          </exclusion>
        </exclusions>
    </dependency>
</dependencies>
```
- 同一管理版本号
  - 在<ptroperties>下写一个标签作为变量
  - 通过${jackson.version}调用
```xml
	<properties>
		<jackson.version>1.1.1</jackson.version>
    <properties>
    <dependencies>
    <!-- 依赖 : spring-context -->
    <dependency>
        <groupId>org.springframework</groupId>
        <artifactId>spring-context</artifactId>
        <!-- 通过${jackson.version}调用 -->
        <version>${jackson.version}</version>
    </dependency>
</dependencies>
<!-- maven编译等命令实际执行的是插件，如果插件版本不匹配可在pom中依赖同级位置导入插件 -->
	<build>
		<plugins>
			<plugin>
				<groupId>org.apache.maven.plugins</groupId>
				<artifactId>maven-compiler-plugin</artifactId>
				<version>3.11.0</version>
			</plugin>
		</plugins>
	</build>

```
# 生命周期
- 一共三套生命周期
 - clean：清理工作。
 - default：核心工作。如：编译、测试、打包、安装、部署等。
 - site：生成报告、发布站点等
- 主要关注clean -->  compile --> test --> package  --> install
- 在同一套生命周期中，我们在执行后面的生命周期时，前面的生命周期都会执行。如package会执行compile和test
- 但在运行install的时候。clean不会运行，因为clean属于clean生命周期，install属于default生命周期，它们不属于同一套生命周期

# 继承与聚合
# 待定
# 依赖传递与冲突
- 依赖传递：导入依赖时，依赖的依赖会自动导入
- 依赖冲突：有重复的依赖导入，会终止依赖传递

- 发生依赖冲突<br>
A - B 1.0 <br>
C - B 2.0 <br>
导入C时，有重复依赖B，发生冲突<br>

- 依赖冲突解决原则<br>
第一原则：谁短谁优先，引用的路径长度<br>
A - C - B 1.0 <br>
F - B 2.0<br>
最后的引入：A C F B的2.0<br>

第二原则：谁上谁优先，依赖声明先后顺序<br>
A - B 1.0 <br>
C - B 2.0 <br>
最后的引入 A C B的1.0<br>
- 依赖冲突可能会导致发生冲突的依赖后面的依赖不被导入
# (待定)
# 私服
