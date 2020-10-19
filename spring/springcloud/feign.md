- Feign：Feign是一个声明式WebService客户端。使用Feign能让编写Web Service客户端更加简单, 它的使用方法是定义一 个接口，然后在上面添加注解<br>
Feign旨在使编写Java Http客户端变得更容易。 前面在使用Ribbon+RestTemplate时，利用RestTemplate对http 请求的封装处理，形成了一套模版化的调用方法。但是在实际开发中，由于对服务依赖的调用可能不止一处，往往 一个接口会被多处调用，所以通常都会针对每个微服务自行封装一些客户端类来包装这些依赖服务的调用

- 使用
    - 依赖
    ```xml
    <dependency>
    <groupId>org.springframework.cloud</groupId>            
    <artifactId>spring-cloud-starter-openfeign</artifactId>
    </dependency>
    ```
    - 编写service家长@FeignClient()注解，参数就是微服务名称
    ```java
    @FeignClient("SERVER-POWER")
    public interface PowerServiceClient {
    @RequestMapping("/power.do") 
    Object power();
    }
    ```
    - 调用
    ```java
    @Autowired
    PowerServiceClient powerServiceClient;

    @RequestMapping("/feignPower.do") 
    public Object feignPower(){
        return powerServiceClient.power(); 
    }
    ```
    
- Feign集成了Ribbon
    