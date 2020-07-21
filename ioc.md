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
    ```