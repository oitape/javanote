- Zuul：对请求的路由和过滤两个最主要的功能
    - 路由功能负责将外部请求转发到具体的微服务实例上，是实现外部访问统一入口的基础
    - 过滤器功能则负责对 请求的处理过程进行干预，是实现请求校验、服务聚合等功能的基础
    - Zuul和Eureka进行整合，将Zuul自身注册为Eureka服务治理下的应用，同时从Eureka中获得其他微服务的消息， 也即以后的访问微服务都是通过Zuul跳转后获得

- 使用
    - 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>             
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-zuul</artifactId> 
    </dependency>
    ```
    - 配置
    ```yaml
    server:
        port: 9000
    eureka:
        client:
          serviceUrl:
              defaultZone: http://localhost:3000/eureka/
        instance:
          instance-id: zuul-1 prefer-ip-address: true
    spring:
        application:
            name: zuul
    ```
    - 启动类
    ```java
    @SpringBootApplication
    @EnableZuulProxy
    public class AppZuul {
        public static void main(String[] args) {                  
            SpringApplication.run(AppZuul.class);
        } 
    }
    ```
    - zuul项目配置
    ```yaml
    zuul:
        prefix: /api   ##路径统一加前缀
        strip-prefix: false
        ignored-services: "*" ##禁用所有可通过serviceId路径的访问
        routes:
            mypower:
                ## eurka注册的服务名
                serviceId: server-power  
                ## 隐藏原服务名  用·power·替换请求路径中的·server-power·
                path: /power/**  
            myorder:
                serviceId: server-order 
                path: /order/**
    ```
    
- Zuul过滤器：如果要编写一个过滤器，则需继承ZuulFilter类 实现其中方法:
    过滤器(filter)是zuul的核心组件 zuul大部分功能都是通过过滤器来实现的。 zuul中定义了4种标准过滤器类型，这 些过滤器类型对应于请求的典型生命周期。             
    - PRE:这种过滤器在请求被路由之前调用。可利用这种过滤器实现身份 验证、在 集群中选择请求的微服务、记录调试信息等。 
    - ROUTING:这种过滤器将请求路由到微服务。这种过滤器 用于构建发送给微服 务的请求，并使用 Apache HttpCIient或 Netfilx Ribbon请求微服务 
    - POST:这种过滤器在路由 到微服务以后执行。这种过滤器可用来为响应添加标准 的 HTTP Header、收集统计信息和指标、将响应从微服务 发送给客户端等。 
    - ERROR:在其他阶段发生错误时执行该过滤器。
    
    