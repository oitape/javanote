- Hystrix：能够保证在一个依赖出问题的情况下，不会导致整体服务失败，避免级联故障，以提高分 布式系统的弹性。<br>“断路器”本身是一种开关装置，当某个服务单元发生故障之后，通过断路器的故障监控(类似熔断保险丝)，向调 用方返回一个符合预期的、可处理的备选响应(FallBack)，而不是长时间的等待或者抛出调用方无法处理的异 常，这样就保证了服务调用方的线程不会被长时间、不必要地占用，从而避免了故障在分布式系统中的蔓延，乃至 雪崩。

- 使用
    - 配置
    ```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>     
        <artifactId>spring-cloud-starter-netflix-hystrix</artifactId>
    </dependency>
    ```
    - 配置
    ```yaml
    hystrix:
         command:
           default:
             execution: 
               isolation:
                 thread:
                   timeoutInMilliseconds: 20000  ## 设置的时间区间内发生多少次就降级限流
             circuitBreaker:
               requestVolumeThreshold: 5
               sleepWindowInMilliseconds : 5000   

         threadpool:
            default:
               coreSize: 10
            power: ## 不同接口配置不同的线程数
               coreSize: 10

    ```
    - 启动类加入注解 **@EnableHystrix** 或者 **@EnableCircuitBreaker**(他们之间是一个继承关系，2个注解所描述的内容是 完全一样的,然后在我们的controller上面加入注解@HystrixCommand(fallbackMethod就是我们刚刚说的方法的名字)
    ```java
      @RequestMapping("/feignPower.do")         
      @HystrixCommand(fallbackMethod = "fallbackMethod") 
      public Object feignPower(String name){
        return powerServiceClient.power(); 
      }  
      
       public Object fallbackMethod(String name){ 
         System.out.println(name);
         return R.error("降级信息");
       }
    ```
    
- 熔断
    一个微服务调用多 次出现问题时(默认是10秒内20次当然 这个也能配置)，hystrix就会采取熔断机制，不再继续调用你的方法(会 在默认5秒钟内和电器短路一样，5秒钟后会试探性的先关闭熔断机制，但是如果这时候再失败一次{之前是20次} 那么又会重新进行熔断) 而是直接调用降级方法，这样就一定程度上避免了服务雪崩的问题
    
- 限流
    hystrix通过线程池的方式来管理你的微服务调用，他默认是一个线程池(10大小) 管理你的所有微服务，你可以 给某个微服务开辟新的线程池
    - 使用
```java
 @RequestMapping("/feignOrder.do")     
 @HystrixCommand(fallbackMethod = "fallbackOrderMethod" ,
        threadPoolKey = "order",
        threadPoolProperties ={@HystrixProperty(name = "coreSize",value = "2")
                              ,@HystrixProperty(name = "maxQueueSize",value = "1"})
public Object feignOrder(String name){
    System.out.println(1);
    return restTemplate.getForObject(ORDERURL+"/order.do",Object.class); 
}
```
        - threadPoolKey 就是在线程池唯一标识， hystrix 会拿你这个标识去计数，看线程占用是否超过了， 超过了就会直 接降级该次调用
        - 比如，这里coreSize给他值为2 那么假设你这个方法调用时间是3s执行完， 那么在3s内如果有超过2个请求进来的 话， 剩下的请求则全部降级

- feign整合hystrix
  feign 以前默认是支持hystrix的， 但是在Spring-Cloud Dalston 版本之后就默认关闭了
  - 配置
  ```yaml 
    feign:
      hystrix:
        enabled: true
  ```
  - 使用
  ```java
     @FeignClient(value = "SERVER-POWER",fallback = PowerServiceFallBack.class) 
     public interface PowerServiceClient {  
        @RequestMapping("/power.do")
        public Object power(@RequestParam("name") String name);
    }
    
    //PowerServiceFallBack类
      @Component
      public class PowerServiceFallBack implements PowerServiceClient {
        @Override
        public Object power(String name) { 
              return R.error("测试降级");
        } 
      }
  ```
  
