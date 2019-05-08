#### spring-cloud-config配置中心使用记录

#### *环境依赖*

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
#### *服务端配置*
```
    spring:
        cloud:
            config:
                server: 
                  git:
                    uri: https://github.com/HurryUpWb/spcloud-config
                    username: username 
                    password: password
                    search-paths: gateway    #指定搜索当前respo下的文件夹
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


#### *客户端配置*
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

#### *主动刷新配置*
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

#### *使用消息总线自动刷新配置*
在配置中新服务端和客户端都需要新增依赖和配置
- 新增依赖
```
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-bus-amqp</artifactId></dependency>
```
- 新增配置
```
    spring:
        rabbitmq:
            host: 192.168.10.180
            port: 5672
            username: admin
            password: admin
            virtual-host: bus
    management:  
        endpoint:
            endpoints:
                web:
                    exposure:
                        include: "*"
```
- 手动调用
修改配置push到respo上时，手动调用服务端的刷新配置URL
 /actuator/bus-refresh（2.x的版本都统一放到了actuator中，1.x直接调用/bus/refresh）
- webhook自动刷新
上述手动调用的方式还是比较麻烦，git仓库提供了webhook自动刷新的功能，把刷新配置的URL配置进来，仓库更新的时候微服务就会自动读取新的配置
#### *整体架构图*
![image](https://github.com/HurryUpWb/spcloud-config/blob/master/configbus.jpg)
