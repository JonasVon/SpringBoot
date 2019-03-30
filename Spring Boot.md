# Spring Boot 

------

## Spring Boot 入门

### 1、Spring Boot 简介

-----

​	`Spring Boot` 是为了简化 `Spring` 应用的创建、运行、调试、部署等一系列问题而衍生的产物，自动装配的特性让我们可以更好的关注业务本身而不是外部的 XML 配置文件，我们只需遵循规范，引入相关的依赖就可以轻易的搭建一个 WEB 工程。

​	未接触 `Spring Boot` 之前，搭建一个普通的 `WEB` 工程旺旺需要花费30分钟左右，如果遇到点问题可能还会耽误更长的时间，但使用了 `Spring Boot` 之后，真正体会到了什么叫分分钟搭建一个工程，从此摆脱了 XML和一大堆 jar包 的恐惧。

​	一句话总结 `Spring Boot`：简化 Spring 应用开发；整个 Spring 技术栈大整合；J2EE 开发的一站式解决方案。



### 2、环境准备

-----------

- jdk 1.8 或以上
- maven 3.3 或以上
- IDE 推荐使用 IDEA 
- Spring Boot 2.1.3



### 3、从一个 Hello World 开始

------------

#### 1.创建一个 Maven 工程，JDK选择1.8

![](/images/1553933197835.png)

GroupId 一般为域名逆序，ArtifactId 一般填写项目名：

![1553933321255](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553933321255.png)

进入到项目后 IDEA 在右下方会弹框，点击 Enable Auto-Import 自动导入：

![1553933427755](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553933427755.png)

#### 2. pom.xml 配置相关依赖

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>com.jonas</groupId>
    <artifactId>HelloSpringBoot</artifactId>
    <version>1.0-SNAPSHOT</version>

    <!-- 继承spring-boot项目 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
    </parent>


    <dependencies>
        <!-- 引入web环境的启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
    </dependencies>

</project>
```

#### 3. 编写一个主程序

```java
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

@SpringBootApplication
public class HelloSpringBoot {
    public static void main(String[] args){
        SpringApplication.run(HelloSpringBoot.class,args);
    }
}
```

注解 `@SpringBootApplication` 用于标注一个主程序类，说明这是一个 `Spring Boot` 应用。

#### 4.编写一个 Controller

编写一个 Controller 用于处理 hello 请求，直接往页面中打印 hello world：

![1553934396969](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553934396969.png)

#### 5. 运行主程序类

测试结果：

![1553934510955](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553934510955.png)

包结构如下：

![1553934660621](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553934660621.png)

主程序所在包一定要包含所有类所在的子包，原因在后面解析。



### 4、初探 Hello World

#### 1. pom 文件

##### i.父项目

在 pom 文件中，一上来我们就引入了一个父项目：

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.3.RELEASE</version>
</parent>
```

点击`spring-boot-starter-parent`进入父项目，可以发现，父项目里面还有一个父项目 `spring-boot-dependencies`：

```xml
<parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-dependencies</artifactId>
        <version>2.1.3.RELEASE</version>
        <relativePath>../../spring-boot-dependencies</relativePath>
</parent>
```

在`spring-boot-dependencies`项目中，管理着一大堆 `Spring Boot` 的依赖，这是 `Spring Boot` 的版本仲裁中心，以后我们导入的依赖默认不需要写版本，因为版本的控制已经交给 `Spring Boot` 了（当然，如果你引入的依赖没有在 dependencies 里面管理着自然需要声明版本号）。

##### ii.启动器

回到 pom 文件中，除了引入了父项目以外，还定义了一个依赖：

```xml
<dependencies>
        <!-- 引入web环境的启动器 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>
</dependencies>
```

在没有使用 `Spring Boot` 之前，我们如果需要 web 环境则需要我们去引入相关的 jar 包或者依赖配置，然而 Spring Boot 将这里所谓的环境进行了一个整合，将所有的功能场景都抽取出来，做成一个个 `starters`（启动器），只需要在项目里面引入这些  `starter` ，相关场景的所有依赖都会导入近来。也就是说，用什么功能就导入什么场景的启动器。具体到这个例子，`spring-boot-starter-web`启动器就帮我们导入了web场景所需要的所有依赖。

#### 2.主程序类

```java
@SpringBootApplication
public class HelloSpringBoot {
    public static void main(String[] args){
        SpringApplication.run(HelloSpringBoot.class,args);
    }
}
```

`@SpringBootApplication` 用于标识在某个类上说明这个类是 `Spring Boot` 的主配置类，`Spring Boot` 就应该运行这个类的 `main` 方法来启动 `Spring Boot` 应用。

点击 `@SpringBootApplication` 注解进入源码看到：

![1553936103197](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553936103197.png)

`@SpringBootConfiguration` ：`Spring Boot` 的配置类，标注在一个类上，表示一个配置类（Spring Boot 将配置也抽取成一个类了）。

`@EnableAutoConfiguration` ：开启自动配置功能。以前我们需要配置的东西，Spring Boot 帮我们进行了自动配置，这个注解就是告诉 Spring Boot 开启自动配置功能。

![1553936439912](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553936439912.png)

`@AutoConfigurationPackage` ：自动配置包。点击进入注解：

![1553936529868](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553936529868.png)

`@Import(AutoConfigurationPackages.Registrar.class)` ：给容器导入一个组件。emmm，那么这里导入的是什么东西呢？点击 `Registrar` 进入看看：

![1553937205912](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553937205912.png)

通过计算发现，导入的包名就是我们定义的主程序类所在的包！！！

**也就是说，`@Import` 的作用就是将主配置类（使用 `@SpringBootApplication` 标注的类）所在包以及下面所有子包里面的所有组件扫到 Spring 容器中！！**

这也就说解析了前面提到的要将控制器类所在的包作为主程序类包的子包，因为它会默认扫描啊，如果放到主程序包的外面那么你的控制器就无法处理请求了，页面就是恶心的 404 了~~



### 5、一键创建 Spring Boot 工程

嘿嘿，直接上图：

![1553938280150](C:\Users\jonas\AppData\Roaming\Typora\typora-user-images\1553938280150.png)

然后填写 GAV 等信息就创建成功了，通过这种方式创建的我们就不需要手动在 pom 里面引入父项目和启动器了，父项目默认继承了，而启动器在创建前可以通过图形化界面的方式勾选具体的场景，然后在 pom 中就给我们自动引入了，方便多了。
