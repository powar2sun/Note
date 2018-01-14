#   Maven
##  本地依赖
```xml
<dependency>
    ...
    <scope>system</scope>
    <systemPath>${project.basedir}/src/lib/xxx.jar</systemPath>
</dependency>
```
##  依赖传递
Maven传递性机制，能够将依赖模块的依赖自动的引入。
若依赖包又依赖同一个包的不同版本，默认采用就近原则选取最近的依赖包，也可在项目中做最早声明的被判断为依赖的版本
##  Excluded & Optional Dependencies 排除依赖和可选依赖
*   可选依赖

```xml
<dependency>
    ...
    <!--value will be true or false
        ture,will be used
        false will not be used
    -->
    <optional>true</optional>
</dependency>
```

*   排除依赖

```xml
<dependency>
    ...
    <exclusions>
        <exclusion>
            <groupId>xxx</groupId>
            <artifactId>xxx</artifactId>
            ...
        </exclusion>
    </exclusions>
</dependency>
```

##  Dependency Scope
Maven 有三套 classpath

*   编译classpath
*   运行classpath
*   测试classpath

Maven 依赖范围 Scope

*   compile,缺省值，随项目一起发布
*   provided,当JDK 或者一个容器已提供该依赖之后才使用,不会被打包。如tomcat提供了ServletAPI
*   runtime,在运行和测试系统的时候需要，但在编译的时候不需要,不会随项目发布
*   system，你必须显式的提供一个对于本地系统中JAR 文件的路径systemPath，该范围是不推荐使用的

##  Dependency Management
dependencyManagement 元素来提供了一种管理依赖版本号的方式。通常会在一个组织或者项目的最顶层的父POM 中看到dependencyManagement 元素。
使用pom.xml 中的dependencyManagement 元素能让所有在子项目中引用一个依赖而不用显式的列出版本号。Maven 会沿着父子层次向上走，直到找到一个拥有dependencyManagement 元素的项目，
然后它就会使用在这个dependencyManagement 元素中指定的版本号。同时在dependenceManagement种，也可以从外部导入POM文件中的依赖项

```xml
<dependencyManagement>
  <dependencies>
    <dependency>
      <groupId>mysql</groupId>
      <artifactId>mysql-connector-java</artifactId>
      <version>5.1.2</version>
    </dependency>
    ...
    <!-- 导入外部POM文件中的依赖项-->
    <dependency>
        <!-- Import dependency management from Spring Boot -->
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>1.3.0.RC1</version>
        <type>pom</type>
        <scope>import</scope>
    </dependency>
  <dependencies>
</dependencyManagement>
<!--
    子项目里就可以添加mysql-connector时可以不指定版本号
-->
<dependencies>  
  <dependency>  
    <groupId>mysql</groupId>  
    <artifactId>mysql-connector-java</artifactId>  
  </dependency>  
</dependencies>  
```

##  Resource 资源管理
```xml
<project>
 ...
 <build>
   ...
  
   <resources>
    <!--
       对应的结构
           Project
           |-- pom.xml
           `-- src
               `-- my-resources
      -->
     <resource>
       <directory>src/my-resources</directory>
     </resource>
     
     <!--选择忽视文件或目录-->
     <resource>
         <directory>src/my-resources</directory>
         <excludes>
           <exclude>**/*.bmp</exclude>
           <exclude>**/*.jpg</exclude>
           <exclude>**/*.jpeg</exclude>
           <exclude>**/*.gif</exclude>
         </excludes>
     </resource>
     <!--选择包含文件或目录-->
     <resource>
         <directory>src/my-resources2</directory>
         <includes>
           <include>**/*.txt</include>
         </includes>
         <excludes>
           <exclude>**/*test*.*</exclude>
         </excludes>
     </resource>
     
   </resources>
   ...
 </build>
 ...
</project>
```

##  Filter 过滤与文件替换
##  Profile构建不同环境的部署包