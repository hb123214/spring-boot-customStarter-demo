# 自定义starter

**1.这个场景需要使用到的依赖是什么?**

**2.如何编写自动配置**

```
@Configuration //指定这个类是一个配置类
@ConditionalOn xxx //在指定条件成立的情况下自动配置类生效
@AutoConfigureAfter   //指定自动配置类的顺序
@Bean //给容器中添加组件
    
@ConfigurationProperties //结合相关的xxxproperties类来绑定相关的配置
@EnableConfigurationProperties //让xxxproperties生效加入到容器中
    
自动配置类要能加载
将需要启动就加载的自动配置类,配置到META-INF/spring.factories
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
org.springframework.boot.autoconfigure.admin.SpringApplicationAdminJmxAutoConfiguration,\
org.springframework.boot.autoconfigure.aop.AopAutoConfiguration,\
```

**3.模式:**

启动器之用来做依赖导入;

专门来写一个自动配置模块;

启动器依赖自动配置;别人只需要引入启动器(starter)

命名: mybatis-pring-boot-starter



## 新建一个空项目

### 1.添加一个空的maven模块(启动器)

​	启动器之用来做依赖导入;

### 2.添加一个spring-boot模块(不添加任何依赖)

​	专门来写一个自动配置模块;

pom:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0" xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
   xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 https://maven.apache.org/xsd/maven-4.0.0.xsd">
   <modelVersion>4.0.0</modelVersion>
    <!--springboot项目的父工程-->
   <parent>
      <groupId>org.springframework.boot</groupId>
      <artifactId>spring-boot-starter-parent</artifactId>
      <version>2.2.6.RELEASE</version>
      <relativePath/> <!-- lookup parent from repository -->
   </parent>

    <!--这个项目的基本信息-->
   <groupId>com.home</groupId>
   <artifactId>mystarter-spring-boot-starter-autoconfigurer</artifactId>
   <version>0.0.1-SNAPSHOT</version>
   <name>mystarter-spring-boot-starter-autoconfigurer</name>
   <description>Demo project for Spring Boot</description>

   <properties>
      <java.version>1.8</java.version>
   </properties>

   <dependencies>
      <!--引入spring-boot-starter,所有starter的基本配置-->
      <dependency>
         <groupId>org.springframework.boot</groupId>
         <artifactId>spring-boot-starter</artifactId>
      </dependency>
   </dependencies>

</project>
```

### 3.starter添加自动配置模块的依赖

​	启动器依赖自动配置模块

pom:

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.home</groupId>
    <artifactId>mystarter-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>

    <dependencies>
        <!--引入自定义starter-->
        <dependency>
            <groupId>com.home</groupId>
            <artifactId>mystarter-spring-boot-starter-autoconfigurer</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    </dependencies>
</project>
```

### 4.添加需要处理逻辑的代码:



```java
public class HelloService {

    HelloProperties helloProperties;

    public String sayHello(String name){
        return helloProperties.getPrefix() + " " + name + " " + helloProperties.getSuffix();
    }

    public HelloProperties getHelloProperties() {
        return helloProperties;
    }

    public void setHelloProperties(HelloProperties helloProperties) {
        this.helloProperties = helloProperties;
    }
}
```

### 5.属性绑定配置类:

```java
//properties绑定类
@ConfigurationProperties(prefix = "mystarter.hello")
public class HelloProperties {
    //前缀
    private String prefix;
    //后缀
    private String suffix;

    public String getPrefix() {
        return prefix;
    }

    public void setPrefix(String prefix) {
        this.prefix = prefix;
    }

    public String getSuffix() {
        return suffix;
    }

    public void setSuffix(String suffix) {
        this.suffix = suffix;
    }
}
```

### 6.自动配置类

```java
@Configuration //自动配置类
@ConditionalOnWebApplication //web应用才成效
@EnableConfigurationProperties(HelloProperties.class)//添加属性绑定器
public class HelloServiceAutoConfiguration {

    //引入属性绑定类
    @Autowired
    HelloProperties helloProperties;

    //给容器中添加组件
    @Bean
    public HelloService helloService(){
        HelloService helloService = new HelloService();
        helloService.setHelloProperties(helloProperties);
        return helloService;
    }
```

### 7.让自动配置类生效

添加文件:

在resource文件下添加META-INF目录并添加spring.factories文件.

添加org.springframework.boot.autoconfigure.EnableAutoConfiguration配置

![image-20200401152442017](C:\Users\10434\AppData\Roaming\Typora\typora-user-images\image-20200401152442017.png)

参考:

springboot的任意autoconfigure

![image-20200401152517033](C:\Users\10434\AppData\Roaming\Typora\typora-user-images\image-20200401152517033.png)

## 测试

测试代码:

```java
@Autowired
HelloService helloService;

@RequestMapping("/mystarter")
@ResponseBody
public String mystarter() {
    return helloService.sayHello("huangbin");
}
```

添加配置:

```yaml
mystarter:
  hello:
    prefix: 你好
    suffix: hello word
```

pom:

```xml
<!--整合自定义starter-->
<dependency>
    <groupId>com.home</groupId>
    <artifactId>mystarter-spring-boot-starter</artifactId>
    <version>1.0-SNAPSHOT</version>
</dependency>
```

