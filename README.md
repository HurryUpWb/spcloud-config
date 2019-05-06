#### spring-cloud-config配置中心

##### *环境依赖*

* 服务端依赖
 
```
    <dependency>    
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-config-server</artifactId>
     </dependency>
```
* 客户端依赖
```
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
```
##### *服务端配置*
```
    spring:
        cloud:
            config:
                server: 
                  git:
                    uri: https://github.com/HurryUpWb/spcloud-config
                    username: username 
                    password: password
```
此时如果配置没有问题则，配置中心服务端已经搭建完成，可以尝试访问github上的配置文件，访问规则是由spring-cloud-config预定义
> * /{application}/{profile}[/{label}]
> * /{application}-{profile}.yml
> * /{label}/{application}-{profile}.yml
> * /{application}-{profile}.properties
> * /{label}/{application}-{profile}.properties

例如，我有一个配置文件名为 h3-config-dev.yml，那么对应的访问url则为
> * /h3-config/dev
> * /h3-config-dev.yml
> * /master/h3-config-dev.yml


##### *客户端配置*
```
    spring:
        application:
         name: h3-config 
         #必须和配置文件名称相同，例如git上的配置文件是 h3-config-dev.yml ,此处的application-name为h3-config
       cloud:
        config:
            uri: http://localhost:8099
            profile: dev
        profiles:
            active: dev   #必须指定启动环境
```
客户端使用配置中心有几个注意点：
>1. spring.application.name 必须和需要使用的配置文件的名称一致
>2. 当前情况下只能配合 actuator 主动刷新实现配置的动态更新

##### *主动刷新配置*
需添加新的依赖
```
    <dependency>
        <groupId>org.springframework.boot</groupId>    
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
```
并新增配置
```
  management:
    endpoint:
        endpoints:
            web:
                exposure:
                    include: "*"
```
使用  /actuator/refresh 便可刷新配置
