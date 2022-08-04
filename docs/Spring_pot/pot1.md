# SpringCloud 项目依赖

********

## 依赖文件

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <modelVersion>4.0.0</modelVersion>

    <groupId>org.example</groupId>
    <artifactId>SCtest</artifactId>
    <packaging>jar</packaging>
    <version>1.0-SNAPSHOT</version>

<!--    官方文件中的 spring-cloud.version 是HOXTON.SR8-->
<!--    对应后文中的 ${spring-cloud.version} 可不能掉了-->
    <properties>
        <maven.compiler.source>11</maven.compiler.source>
        <maven.compiler.target>11</maven.compiler.target>
        <spring-cloud.version>2021.0.3</spring-cloud.version>
    </properties>

<!--    许多web注解的依赖，如果没有特殊需求，建议所有spring项目都加上-->
    <dependencies>
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
        </dependency>

<!--        这里有一个服务端依赖和客户端依赖，有的时候两个一起可以，有的时候不行，-->
<!--        建议服务端只加服务端依赖，毕竟服务端可以实现客户端功能-->
        <dependency>
            <groupId>org.springframework.cloud</groupId>
            <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
        </dependency>
    </dependencies>
    
<!--    此依赖管理器可以拆分为几个子依赖写入dependencies，但是没必要，建议加上-->
<!--    type 属性常用的有 pom 和 jar 两种，-->
<!--    jar 直接打包所有资源-->
<!--    pom 如果没有官方的 sping-cloud-starter-config 依赖，pom默认不会打包.yml等配置资源-->
<!--    并且很多包没有jar打包版本，如果遇到必须使用jar打包方式，可以考虑先使用pom下载依赖，然后再用-->
<!--    jar构建项目-->
    <dependencyManagement>
        <dependencies>
            <dependency>
                <groupId>org.springframework.cloud</groupId>
                <artifactId>spring-cloud-dependencies</artifactId>
                <version>${spring-cloud.version}</version>
                <type>pom</type>
                <scope>import</scope>
            </dependency>
        </dependencies>
    </dependencyManagement>
    
<!--    这个东西在springcloud官方文档里没有，但是强调了springcloud是依赖于springboot的-->
<!--    因此springcloud项目也应该继承这个父项目-->
<!--    然而依然有可以直接继承的springcloud父项目，叫做spring-cloud-starter-parent-->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.7.2</version>
    </parent>

</project>
```

<br>

## 其他可能用到的依赖

*********

这个依赖解决了构建cloud项目不包含配置资源的问题
```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-config</artifactId>
		</dependency>
```

*********

这个依赖加入主要是 Netflix 中默认使用的是 Ribbon 的负载均衡工具，
虽然说 eureka 自带负载均衡，但是不排除高版本，特别是加入naxcon后负载均衡无法使用情况
如果出现问题，可以加上
```xml
		<dependency>
			<groupId>org.springframework.cloud</groupId>
			<artifactId>spring-cloud-starter-loadbalancer</artifactId>
		</dependency>
```

*********

