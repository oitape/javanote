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
         name: test
     cloud:
       config:
         server: git:
            uri: https://github.com/513667225/my-spring-cloud-config.git #配置文件在github上的地址
            username:
            password:
            # search-paths: foo,bar* #Configserver会在 Git仓库根目录、 foo子目录，以及所有以
bar开始的子目录中查找配置文件。
            # clone-on-start: true #启动时就clone仓库到本地，默认是在配置被首次请求时，config server才会clone git仓库
         #native:
               #search-locations: classpath:/config #若配置中心在本地，本地的地址
```
- 启动配置中心服务器
```java
@SpringBootApplication
@EnableConfigServer
public class AppConfig {
   public static void main(String[] args) {          
      SpringApplication.run(AppConfig.class);
   } 
}
```


-  客户端使用
   - 依赖
   ```xml
   <dependency> 
        <groupId>org.springframework.cloud</groupId> 
        <artifactId>spring-cloud-starter-config</artifactId>
   </dependency>
   ```
   
   
