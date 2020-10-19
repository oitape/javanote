- 服务端
    - 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
    </dependency>
    ```
    - 配置
    ```yaml
    server:
      port: 3001
    eureka:
        server:
            enable-self-preservation: false  #关闭自我保护机制
            eviction-interval-timer-in-ms: 4000 #设置清理间隔（单位：毫秒 默认是60*1000）
        instance:
            hostname: eureka3001.com

        client:
          #    registerWithEureka: false #不把自己作为一个客户端注册到自己身上
          #    fetchRegistry: false  #不需要从服务端获取注册信息（因为在这里自己就是服务端，而且已经禁用自己注册了）
           serviceUrl:
              defaultZone: http://eureka3000.com:3000/eureka,http://eureka3002.com:3002/eureka
               
    ```
    - 启动
    ```java
    @EnableEurekaServer
    @SpringBootApplication
    public class AppEureka {
        public static void main(String[] args) {
            SpringApplication.run(AppEureka.class);                     
        } 
    }    
    ```
- 客户端
    - 依赖
    ```xml
     <dependency>
        <groupId>org.springframework.cloud</groupId>               
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    ```

    - 配置
    ```yaml
    server:
      port: 6000
    eureka:
        client:
        serviceUrl:
            defaultZone: http://localhost:3000/eureka/  #eureka服务端提供的注册地址 参考服务端配置的这个路径
        instance:
            instance-id: power-0 #此实例注册到eureka服务端的唯一的实例ID
            prefer-ip-address: true #是否显示IP地址
            leaseRenewalIntervalInSeconds: 10 #eureka客户需要多长时间发送心跳给eureka服务器，表明它仍然活着,默认为30 秒 (与下面配置的单位都是秒)
            leaseExpirationDurationInSeconds: 30 #Eureka服务器在接收到实例的最后一次发出的心跳后，需要等待多久才可以将此实例删除，默认为90秒

    spring:
        application:
            name: server-power #此实例注册到eureka服务端的name
    ```
    - Application添加注解@EnableEurekaClient