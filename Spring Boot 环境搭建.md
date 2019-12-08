# Spring Boot 环境搭建 #

[toc]

## 安装MySql数据库 ##

第一步，进入官网[MySql](https://www.mysql.com/)，下载安装[MySql Community版](https://dev.mysql.com/downloads/mysql/)，不要下载MySQL Enterprise版。

第二步，解压安装包，进入解压后的目录，新建文件my.ini（本步可省略）：
```ini
[mysqld]
# 设置3306端口
port=3306
# 设置mysql的安装目录
basedir=C:\Program Files\MySQL
# 设置mysql数据库的数据的存放目录
datadir=C:\Program Files\MySQL\Data
# 允许最大连接数
max_connections=200
# 允许连接失败的次数。
max_connect_errors=10
# 服务端使用的字符集默认为utf8mb4
character-set-server=utf8mb4
# 创建新表时将使用的默认存储引擎
default-storage-engine=INNODB
# 默认使用“mysql_native_password”插件认证
#mysql_native_password
default_authentication_plugin=mysql_native_password
# 设置东8时区（北京时间）
default-time-zone = '+8:00'
[mysql]
# 设置mysql客户端默认字符集
default-character-set=utf8mb4
[client]
# 设置mysql客户端连接服务端时默认使用的端口
port=3306
default-character-set=utf8mb4
```

第二部，以管理员身份运行控制台，进入mysql的目录下，执行如下代码，代码执行完毕之后，**控制台会输出一串临时密码，记住这个密码**:

```bat
cd .\bin
mysqld --initialize --console # 初始化数据库
```

第四步，以管理员身份运行控制台，进入mysql的目录下，执行如下代码:

```bat
cd .\bin
.\mysqld.exe --install mysql # 将mysql添加到系统服务中
net start mysql # 启动mysql服务
.\mysql.exe -uroot -p # 执行完命令之后，需要输入第二步中生成的临时密码，数据库连接成功
```

第五步，在mysql.exe中输入如下SQL语句，重置密码，mysql安装成功:

```sql
ALTER USER root@localhost IDENTIFIED BY '123456';
```
## 安装Maven ##

第一步，进入[Apache Maven Project](https://maven.apache.org/)中，下载[ZIP版压缩包](http://mirrors.tuna.tsinghua.edu.cn/apache/maven/maven-3/3.6.3/binaries/apache-maven-3.6.3-bin.zip)。

第二部，解压压缩包，进入解压后的目录（已`D:\apache-maven-3.6.3\`为例），新建目录repository。

第三步，打开`D:\apache-maven-3.6.3\conf\settings.xml`，在`<settings>`标签下添加如下代码：
```xml
<!-- 确定本地仓库路径，就是第一步新建的repository目录 -->
<localRepository>D:\apache-maven-3.6.3\repository</localRepository>
```
在`<mirrors>`标签下添加如下代码：
```xml
<!-- 指向淘宝中心仓库 -->
<mirror>
    <id>maven-public</id>
    <mirrorOf>*</mirrorOf>
    <name>maven-public</name>
    <url>http://maven.aliyun.com/nexus/content/groups/public/</url>
</mirror>
```

第四步，将`D:\apache-maven-3.6.3\`添加至环境变量Paht中，如果使用集成开发环境，可忽略此步。

## 构建Spring Boot工程 ##

使用集成开发快发环境可以快速构建工程。如果使用eclipse开发，需要下载Spring Tool插件。如果使用idea开发，要选择构建maven工程，而不是Spring工程。

也可以使用maven直接构建工程，然后倒入到集成开发环境中。

工程目录结构如下：

* src 源码目录
  * main 生产代码存放目录
    * java Java代码存放目录
    * resources 资源文件存放目录
  * test 测试代码存放目录
    * java 测试java代码存放目录
    * resources 测试资源代码存放目录
* pom.xml maven构建文件
* Readme.md 工程说明文档
* Dockerfile docker镜像构建文件

## 修改工程中的pom.xml文件 ##

maven通过pom.xml构建整个工程，每个java项目都有一个pom.xml文件。pom.xml中配置了工程的名称，依赖库，编译方式，打包方式等信息。pom的书写格式如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<project xmlns="http://maven.apache.org/POM/4.0.0"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 http://maven.apache.org/xsd/maven-4.0.0.xsd">
    <!-- 模型版本，maven 2和maven 3的模型版本为4.0.0，maven 4的模型版本为4.1.0 -->
    <modelVersion>4.0.0</modelVersion>

    <!-- 项目组ID，每个maven工程都有一个组ID -->
    <groupId>com.test</groupId>
    <!-- 项目ID，每一个maven工程都有一个项目ID -->
    <artifactId>spring-boot-template</artifactId>
    <!-- 版本号 -->
    <version>1.0-SNAPSHOT</version>
    <!-- 项目名称，非必填项 -->
    <name>spring-boot-template</name>
    <!-- 项目链接，非必填 -->
    <url>http://127.0.0.1:10000/</url>
    <!-- 项目描述，非必填 -->
    <description>Spring Boot 模板工程</description>

    <!-- 项目开发者信息，非必填 -->
    <developers>
        <developer>
            <name>Zhao Jinyan</name>
            <email>949474952@qq.com</email>
            <organization>hx</organization>
        </developer>
    </developers>

    <prope

    <!-- Spring Boot项目需要继承于spring-boot-starter-parent，当然可以不继承，采用其他方法 -->
    <parent>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-parent</artifactId>
        <version>2.1.5.RELEASE</version>
    </parent>

    <!-- 配置项目依赖 -->
    <dependencies>
        <!-- 要构建web项目，必须引用此项目 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-web</artifactId>
            <!-- 因为继承了spring-boot-starter-parent，所有spring-boot开头的依赖库不需要添加版本号 -->
        </dependency>
        <!-- JPA 依赖，如果不使用jpa，不需要这个依赖配置 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-data-jpa</artifactId>
        </dependency>
        <!-- Spring boot单元测试 -->
        <dependency>
            <groupId>org.springframework.boot</groupId>
            <artifactId>spring-boot-starter-test</artifactId>
            <scope>test</scope>
        </dependency>
        <!-- 单元测试框架 -->
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>4.12</version>
            <scope>test</scope>
        </dependency>
        <!-- lombok，用于简化代码，需要配合ide插件 -->
        <dependency>
            <groupId>org.projectlombok</groupId>
            <artifactId>lombok</artifactId>
            <version>1.18.8</version>
            <scope>provided</scope>
        </dependency>
        <!-- Swagger用于生成Web api -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger2</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- 为Swagger提供一个html页面 -->
        <dependency>
            <groupId>io.springfox</groupId>
            <artifactId>springfox-swagger-ui</artifactId>
            <version>2.9.2</version>
        </dependency>
        <!-- MySql JDBC -->
        <dependency>
            <groupId>mysql</groupId>
            <artifactId>mysql-connector-java</artifactId>
            <version>8.0.15</version>
        </dependency>
    </dependencies>

    <!-- Spring Boot 打包插件 -->
    <build>
        <!-- 打包后的名称，可不指定 -->
        <finalName>spring-template</finalName>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <configuration>
                    <!-- 指定启动类 -->
                    <mainClass>com.test.Application</mainClass>
                </configuration>
                <executions>
                    <execution>
                        <goals>
                            <!--subppress MybatisMapperXmlInspection -->
                            <goal>repackage</goal>
                        </goals>
                    </execution>
                </executions>
            </plugin>
        </plugins>
    </build>

    <!-- Spring官方仓库，可不设置 -->
    <repositories>
        <repository>
            <id>spring-milestones</id>
            <name>Spring Milestones</name>
            <url>https://repo.spring.io/libs-milestone</url>
            <snapshots>
                <enabled>false</enabled>
            </snapshots>
        </repository>
    </repositories>
</project>
```

## Spring Boot配置文件 ##

Spring Boot默认需要一个配置，配置存放于`src\main\resources`，名称为`bootstrap.yml`或`application.yml`，bootstrap和application有所区别，bootstrap在application之前加载。

```yml
server:
  port: 10000 # 端口
spring:
  application:
    name: retard-oauth # 应用名称
  datasource:
    url: jdbc:mysql://localhost:3306/retard?useUnicode=yes&characterEncoding=UTF-8&serverTimezone=GMT%2B8 # 数据库URL
    username: root # 数据库用户名
    password: 123456 # 数据库密码
    driver-class-name: com.mysql.cj.jdbc.Driver # JDBC类
    type: com.zaxxer.hikari.HikariDataSource # 数据连接池类
    hikari:
      minimum-idle: 5
      maximum-pool-size: 15
      auto-commit: true
      idle-timeout: 30000
      pool-name: DatebookHikariCP
      max-lifetime: 1800000
      connection-timeout: 30000
      connection-test-query: SELECT 1
  output:
    ansi:
      enabled: always # 控制台测色显示
  jpa:
    hibernate:
      ddl-auto: update # jpa数据表更新模式
    show-sql: true # 显示sql
    open-in-view: false
    database-platform: org.hibernate.dialect.MySQL8Dialect # 指定sql方言
```

## Spring Boot 启动类 ##

启动类是Spring Boot的程序入口和配置入口，具体代码如下：

```java
package com.test.Application;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;

/**
 * 程序入口.
 * @author Zhao Jinyan
 */
@SpringBootApplication(scanBasePackages = "com.test")
public class Application {
	public static void main(String[] args) {
		SpringApplication.run(Application.class, args);
	}
}
```

`@SpringBootApplication`是Spring Boot的启动注解，`scanBasePackages`配置Spring Boot的包扫描路径，可以不配置。

## Swagger 2 配置 ##



## Lombok 使用方法 ##

Lombok通过注解简化代码，但是需要IDE下载插件。

@Data: 标注于类上，相当于@ToString\@EqualsAndHashCode\@Getter\@Setter\@RequiredArgsConstructor\@AllArgsContructor\@NoArgsContructor的组合
@ToString: 重载toString方法
@EqualsAndHashCode: 重载equals和hashCode方法
@RequiredArgsConstructor: 生成必填属性(final和@NonNull)构造方法
@AllArgsContructor: 全属性构造方法
@NoArgsContructor: 无参构造方法


## Windows永久修改环境变量的三种方法 ##

第一种，就是通过图形界面修改，这里不做详细介绍。

第二种，通过wmic修改环境变量(需要管理员权限)，具体如下:

```bat
rem 查看path环境变量的用户和值
wmic ENVIRONMENT where name="PATH" get UserName,VariableValue
rem 修改path
wmic ENVIRONMENT where "name='PATH' and username='<system>'" set VariableValue="%PATH%;T:\maven\bin"
rem 新建环境变量
wmic ENVIRONMENT create name="MAVEN_HOME",username="<system>",VariableValue="D:\maven"
rem 删除环境变量
wmic ENVIRONMENT where "name='MAVEN_HOME' and username='<system>'" delete
```
第三种，通过setx修改环境变量，有些PC没有安装setx命令，需要到Microsoft官网下载，具体如下：

```bat
rem 新建环境变量
setx MAVEN_HOME D:\maven /M
rem 修改环境变量
setx Path %Path%;D:\maven\bin /M
rem 删除环境变量
setx MAVEN_HOME "" /M
```
