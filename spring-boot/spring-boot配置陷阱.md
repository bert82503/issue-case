

spring-boot配置陷阱
===============
> 钟伦甫/巨石，2016.08.26


spring-boot 官方文档就 properties 文件加载有如下说明<sup>[1][1]</sup>：
> #### 24.3 Application property files/应用属性文件
> SpringApplication will load properties from application.properties files in the following locations and add them to the Spring Environment:
>
> 1. A /config subdirectory of the current directory
> 2. The current directory
> 3. A classpath /config package
> 4. The classpath root
>
> The list is ordered by precedence (properties defined in locations higher in the list **override** those defined in lower locations).

简言之，properties 文件有四处加载位置，
如果多处都存在`application.properties`，且 property 有重复，则前者会**覆盖**后者。

> 我写了个项目 [zhongl/spring-boot-config-trap](https://github.com/zhongl/spring-boot-config-trap) 可以演示这一特性：
>
> 通过分别运行`tag:resources`和`tag:config`两版本的代码，
发现 http://localhost:8080 结果因`config/application.properties`文件内容的变化而不同。

**在特定环境中部署运行时，通过这一特性可以灵活的调整某些 property 的初始值（即`resources/application.properties`中的预设值）。**


### 陷阱
可惜的是，`在不恰当的开发习惯下，也容易引发配置疏忽大意，导致错误的 properties 生效而造成严重的系统故障。`

> 比如，我在`resources/application.properties`中配置 JDBC 参数是指向非生产环境的数据库，方便日常开发测试。
> 到生产环境中部署时，通过`config/application.properties`来调整 JDBC 参数的指向。
> `一旦忘记更改指向，服务是可以正常启动的，而错误直到数据读写出现才会暴露`，而一切为时已晚……


### 规避
1. 本地开发也使用`config／application.properties`，而非`resources/application.properties`
2. `.gitignore` 添加 `config`，避免本地配置扩散到其他环境


### 启发
1. 依赖环境的配置不能有默认值
2. `配置可能引发的问题，一定要在启动阶段暴露`，哪怕是无法提供服务，总比提供错误的服务要好


[1]: http://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html#boot-features-external-config-application-property-files


[原文](https://zhongl.github.io/2016/08/26/spring-boot-config-trap/)

