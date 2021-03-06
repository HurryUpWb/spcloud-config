#### spring-cloud-config配置中心

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
    <artifactId>spring-cloud-starter-bus-amqp</artifactId>
 </dependency>
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
上述手动调用的方式还是比较麻烦，git仓库提供了webhook自动刷新的功能，github上的配置地方如图
![imgage](https://github.com/HurryUpWb/spcloud-config/blob/master/hook-setiing.png)
把刷新配置的URL配置进来，仓库更新的时候微服务就会自动读取新的配置

#### *整体架构图*
![imgage](https://github.com/HurryUpWb/spcloud-config/blob/master/configbus.jpg)


#### *将配置放到git私服*

>1. 先定义出一个配置仓库
     git init --bare
>2. 本地克隆这个仓库
     git clone git@192.168.10.60:/home/git/spcloud-config
>3. 将所有文件添加到本地仓库里
     git add .
>4. 提交
     git commit -m '提交的信息' 
>5. push到远程仓库
     git push

注意点：
* 提交时权限问题
  >当前账户为git时,创建远程仓库的时候是root用户，需要把权限给git用户
    ```chown -R git:git spcloud-config/```


* spring.cloud.config.server.git.uri 私服写法
    >git@192.168.10.60:/home/git/spcloud-config


#### *git私服配置hooks*
由于需要自动刷新配置，借助git的hook机制，在更新了配置push到远程仓库后自动请求上述‘/bus/refresh’接口实现配置的自动刷新

* 在远程仓库下找到 hooks 文件夹
* 新增名为 post-receive的hook，必须是可运行的脚本文件支持多种格式
 ```  
    #!/bin/bash
    curl -X POST -i "http://192.168.10.174:8099/actuator/bus-refresh"
 ```
 * 更新配置push到仓库时会自动请求这个链接实现刷新
