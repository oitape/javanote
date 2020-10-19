- Ribbon：提供客户端的软件负载均衡算法，在配置文件中 列出Load Balancer(简称LB)后面所有的机器，Ribbon会自动的帮助你基于某种规则(如简单轮询，随机连接 等)去连接这些机器。
- 服务端负载均衡
    - 服务端负载：先经过一个代理服务器(nginx、slb、ha)，然后通过这个代理服务器通过算法(轮询，随机，权重等等..) 反向代理你的服务，来完成负载均衡
    - 客户端负载：客户端的负载均衡则是一个请求在客户端的时候已经声明了要调用哪个服务，然后通过具体的负载均衡算法来完成负载均衡

- Ribbon使用：eureka已经把ribbon集成到他的依赖里面
    - 配置
    ```java
    @Bean
    @LoadBalanced
    public RestTemplate restTemplate(){
        RestTemplate restTemplate = new RestTemplate();
        return restTemplate;
    }
    ```

- 通过服务名调用
```java
 private static final String URL="http://SERVER-POWER";      
 
 @Autowired
 private RestTemplate restTemplate;
 
 @RequestMapping("/power.do") 
 public Object power(){
    return restTemplate.getForObject(URL+"/power.do",Object.class); 
 }
```

- 负载策略
![](/assets/iShot2020-10-19下午08.42.41.png)
指定负载均衡算法
```java
 @Bean
public IRule iRule(){
    return  new RoundRobinRule();
}
```

- 