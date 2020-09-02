* Inversion of control

  * 配置风格
    * schemal-based-------xml
    * annotation-based-----annotation
    * java-based----java Configuration
  * xml配置方式
    ```java
    ClassPathXmlApplicationContext context = new ClassPathXmlApplicationContext("classpath:spring.xml");
      Service service = (Service)context.getBean("service");
    ```
  * 注解配置

    ```java
    @Configuration
    @ComponentScan("top.oitm")
    public class Spring {

    }

    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Spring.class);
    ```

  * 混合配置

    ```java
    @Configuration
    @ComponentScan("top.oitm")
    @ImportResource("classpath:spring.xml")
    public class Spring {

    }    
    AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(Spring.class);
    ```

  * 设置自动装配类型
    ```
    default-autowire="byType"
    ```

* 配置

  * 配置bean的先后顺序
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
  * 相同类型多bean

    ```
    //第一种设置默认使用的bean加上下面注解
    @Primary

    //第二种使用指定bean
    @Qualifier("bean")
    ```

- 扫描

```java
public class AnnotationConfigApplicationContext {

    public void scan(String basePackage){
        String rootPath = this.getClass().getResource("/").getPath();
        String  basePackagePath = basePackage.replaceAll("\\.","\\\\");
        File file = new File(rootPath+"//"+basePackagePath);
        String names[]=file.list();
        for (String name : names) {
            name=name.replaceAll(".class","");
            try {
               Class clazz =  Class.forName(basePackage+"."+name);
                //判斷是否是屬於@servi@xxxx
                if(clazz.isAnnotationPresent(Luban.class)){
                    Service service = (Service) clazz.getAnnotation(Service.class);
                    System.out.println(service.value());
                }

            } catch (Exception e) {
                e.printStackTrace();
            }
        }

    }
}

```
