- Hystrix：（单纯的Hystrix）提供了对于微服务调用状态的监控信息，需要结合spring-boot-actuator模块使用
- 使用
    - 在微服务内部添加依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>             
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
    ```
    - 访问 {$host}/actuator/hystrix.stream便可以看见微服务调用的状态信息，在Spring Finchley 版本以前访问路径是/hystrix.stream，如果是Finchley 的话 还得在微服务项目的yml里面加入配置:
    ```yaml
    management:
          endpoints:
            web:
              exposure:
                include: '*'
    ```
    - 新建HystrixDashbord项目
        - 添加依赖
        ```xml
         <dependency>
            <groupId>org.springframework.cloud</groupId>     
            <artifactId>spring-cloud-starter-netflix-hystrix-dashboard</artifactId>
        </dependency>
        ```
        - 启动类上加入注解
        ```java    
        @EnableHystrixDashboard
        @SpringBootApplication
        public class AppHystrixDashboard {
            public static void main(String[] args) {
                SpringApplication.run(AppHystrixDashboard.class);
            }
        }
        ```
        - 访问
        ```
        http://{hsot}/hystrix
        ```
- 如何查看监控
    - dashbord页面填入监控的地址,会跳到相应监控仪表盘
    ```
    http://localhost:5000/actuator/hystrix.stream
    ```
    - 必须在服务之间调用过了才会展示信息
    - 仪表盘解读
        ![](/assets/iShot2020-10-25下午05.35.22.png)
        - 实心圆:共有两种含义。它通过颜色的变化代表了实例的健康程度，它的健康度从绿色该实心圆除了颜色的变化之外，它的大小也会根据实例的请求流量发生变化，流量越大该实心圆就越大。所以通过 该实心圆的展示，就可以在大量的实例中快速的发现故障实例和高压力实例。
        - 曲线:用来记录2分钟内流量的相对变化，可以通过它来观察到流量的上升和下降趋势。