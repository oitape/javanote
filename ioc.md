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