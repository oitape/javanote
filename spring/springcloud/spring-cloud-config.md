- 依赖
```
<dependency> 
   <groupId>org.springframework.cloud</groupId>    
   <artifactId>spring-cloud-config-server</artifactId>
</dependency>
```
- yml配置

```yaml
server:
  port: 8080
spring:
  application:
    name: config-server

  cloud:
    config:
      server:
        git:
          uri: https://github.com/xxxx.git
          search-paths: foo,bar*  #Configserver会在 Git仓库根目录、 foo子目录，以及所有以 bar开始的子目录中查找配置文件。
          clone-on-start: true  #启动时就clone仓库到本地，默认是在配置被首次请求时，config server才会clone git仓库

eureka:
  client:
    serviceUrl:
        defaultZone: http://localhost:3000/eureka/  #eureka服务端提供的注册地址 参考服务端配置的这个路径
  instance:
    instance-id: config-1 #此实例注册到eureka服务端的唯一的实例ID
    prefer-ip-address: true #是否显示IP地址
    leaseRenewalIntervalInSeconds: 1 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
    leaseExpirationDurationInSeconds: 3 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒               
```
- 启动配置中心服务器
```java
@SpringBootApplication
@EnableConfigServer
@EnableEurekaClient
public class AppConfig {
   public static void main(String[] args) {          
      SpringApplication.run(AppConfig.class);
   } 
}
```
- 通过配置中心读取配置信息
```
http://localhost:8080/test-config.yml
```
- 读取不同环境的配置
```
http://localhost:8080/test-config-dev.yml
```
- config访问配置文件，是需要一个具体的访问规则的，如下
```
  /{application}/{profile}[/{label}] 
  /{application}-{profile}.yml 
  /{label}/{application}-{profile}.yml 
  /{application}-{profile}.properties 
  /{label}/{application}-{profile}.properties
```
application就是配置文件的名字， profile就是对应的环境，  label就是不同的分支 


-  客户端使用
   - 依赖
   ```xml
   <dependency> 
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   ```
   - 启动类和配置服务器一样
   - config读取配置还需要增加一个**bootstrap.yml** 配置文件
   ```yaml
   spring:
        cloud:
          config:
            name: test-config #这是我们要读取的配置文件名 对应获取规则的{application} 
            profile: dev #这个是要获取的环境 对应的便是{profile}
            label: master #这个就是获取的节点 对应的是{label}
            uri: http://localhost:8080/ #这就是我们config server的一个地址
   ```
   
- 高可用
  - config server注册到eureka上，client端也注册到eureka上
  - client的配置文件进行以下改动
  ```yaml
  spring:
      cloud:
        config:
        name: test-config 
        profile: dev 
        label: master 
        discovery:
          enabled: true
          service-id: test-config 
  eureka:
      client:
        serviceUrl:
          defaultZone: http://localhost:3000/eureka/
  ```
   
   
