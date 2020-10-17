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