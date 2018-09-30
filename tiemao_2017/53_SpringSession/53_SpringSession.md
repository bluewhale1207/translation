# Spring Session


Session，中文翻译为 "会话"，在计算机领域，用于描述用户端和服务器之间的通讯。当然，基于Http协议的浏览器在请求Web服务器资源时，我们通过会话(Session)来标识不同的客户端/用户。


## Spring-Session简介

Spring Session 是 [Spring.io](https://spring.io/) 组织维护的一个开源项目，基于 Java Web 的 API，将Web服务的会话状态抽离出来，放到共享存储中。这样就可以很容易将单机Web系统平滑扩展为Web集群系统。

项目的官方网站为: <https://projects.spring.io/spring-session/>

顾名思义，Spring Session 可以用来管理JavaWeb系统的用户会话信息。

其主要功能特征包括:

- 用户Session管理相关的API和具体实现

- HttpSession - 以独立的方式，替换如Tomcat之类Web容器所提供的HttpSession，。

- Clustered Sessions - 集群会话管理, 通过Spring Session，可以很轻易地实现Web服务器集群，不需要根据各种Web容器进行一堆繁琐的配置.

- Multiple Browser Sessions - 支持多个会话同时并存，通过 Spring Session，可以在同一个浏览器中支持多个用户会话状态。 (有点类似Google的多账号登录认证).

- RESTful APIs - 可以在 headers 中指定session ids，来兼容 RESTful APIs

- WebSocket - 在WebSocket调用中保证 HttpSession 存活。

下面，我们通过具体的示例来使用Spring-Session。

## 配置示例

jar包下载/依赖配置。

现在很多Java项目使用MAVEN来构建。 

根据官方文档，`pom.xml`文件中配置的maven依赖如下：

```
<dependencies>
    <dependency>
        <groupId>org.springframework.session</groupId>
        <artifactId>spring-session</artifactId>
        <version>1.3.1.RELEASE</version>
    </dependency>
</dependencies>
```

因为 spring-session 的版本和Spring相互独立，所以版本号可以直接写到 dependency 元素中, 当然, 抽取到 properties 配置项中更好。

如果是其他构建工具，那么打开官方网站，在首页面下方即可看到。

假若需要直接依赖 JAR 包, 可以到 <http://mvnrepository.com/> 网站进行搜索, 例如 `spring-session` 关键字。

结果如下图所示:

![](01_search_mvnrepository.png)

点击对应的连接，看到各种版本的列表页面：

![](02_spring-session-mvn.png)

选择需要的版本，点击版本号，进入该版本对应的页面。

![](03_spring-session-jar.png)

可以看到，其中有 `jar` 相关的链接，点击即可下载。

当然，从中我们也可以看到其他各种MAVEN相关的构建工具的依赖配置，根据需要选择即可。该页面的下方还列出了直接依赖。 建议收藏该网站，以备不时之需。


## 浏览器Cookie技术简介

我们知道, 浏览器有一种叫做 Cookie 的技术。 在请求服务器时, 会将 Cookie 信息自动添加到 http 请求头中。

Cookie 可以由(服务器通过) http 响应头设置, 也可以在网页中通过 JavaScript 代码设置。

本文不对此进行深入讲解, Cookie 的策略...。










## Session 的实现原理

本文讨论的Session是指Http协议下的会话。

只要是能携带唯一身份标识的通信方式，都可以用来实现 Session, 例如, 请求参数, 特定 http请求头,  Cookie 等等。


Session 大多借助于 Cookie 技术来实现。原因是支持度广，实现简单，操作方便。

流程大致如下:

1. 用户在客户端浏览器打开登录页面。
2. 输入用户名和密码, 点击登录按钮;
2.1 浏览器通过HTTP协议发送请求;
2.2 服务器收到请求,校验通过, 缓存该用户信息, 将唯一标识(如UUID)写入Cookie;
2.3 浏览器收到响应, 如果有Cookie则保存到客户端;  根据响应数据,决定是否跳转页面。
3.1 浏览器自动执行跳转, 自动带上所有的 Cookie 信息请求一个新页面。
3.2 服务器收到请求, 校验 Cookie, 如果是合法的数据, 则根据该用户信息返回对应的页面内容。
3.3 浏览器显示对应的页面。

这个过程中可能因为各种异常情况进入不同的流程。






###


标准的MAVEN-Web项目结构如下:

```
project-name/
  ----pom.xml
  ++--target/
  ++--src/
      ++--main/
          ++--java/
          ++--resources/
          ++--webapp/
              ----index.jsp
              ++--WEB-INF/
                  ----web.xml
      ++--test/
          ++--java/
          ++--resources/

```


创建项目 `spring-session-demo`。


pom.xml 文件如下所示:

```
<project 
  xmlns="http://maven.apache.org/POM/4.0.0" 
  xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
  xsi:schemaLocation="http://maven.apache.org/POM/4.0.0 
    http://maven.apache.org/maven-v4_0_0.xsd">

    <modelVersion>4.0.0</modelVersion>
    <groupId>com.cncounter.demo</groupId>
    <artifactId>spring-session-demo</artifactId>
    <packaging>war</packaging>
    <version>1.0-SNAPSHOT</version>
    <name>spring-session-demo Maven Webapp</name>
    <url>http://maven.apache.org</url>
    <dependencies>
        <dependency>
            <groupId>junit</groupId>
            <artifactId>junit</artifactId>
            <version>3.8.1</version>
            <scope>test</scope>
        </dependency>

        <dependency>
            <groupId>org.springframework.session</groupId>
            <artifactId>spring-session</artifactId>
            <version>1.3.1.RELEASE</version>
        </dependency>
    </dependencies>
    <build>
        <finalName>spring-session-demo</finalName>
    </build>
</project>
```






