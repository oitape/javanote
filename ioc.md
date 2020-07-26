- Inversion of control 
    - 配置风格
        - schemal-based-------xml
        - annotation-based-----annotation
        - java-based----java Configuration
    - xml配置方式
    ```java
      ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring.xml");
        Service service = (Service)context.getBean("service");
    ```
    - 注解配置
    ```java
    @Configuration
    @ComponentScan("top.oitm")
    public class Spring {
    
    }
    
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Spring.class);
    ```
    - 混合配置
    ```java
    @Configuration
    @ComponentScan("top.oitm")
    @ImportResource("classpath:spring.xml")
    public class Spring {
    
    }    
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Spring.class);
    ```
    - 设置自动装配类型
    ```
    default-autowire="byType"
    ``` 
- 配置
    ```java
    public class PropertyConfig implements InitializingBean {
    public PropertyConfig() {
        System.out.println("1、ConfigProperty");
    }
    @PostConstruct
    public void init() {
        System.out.println("2、init");
    }
    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("3、afterPropertiesSet");
    }
    @PreDestroy
    public void preDestroy(){}
    }
    ```