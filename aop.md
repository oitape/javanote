- 使用
    ```java
    @Component
    @Aspect
    public class AspectJ {
        /**
         * aop下所有宝所有类所有方法任意返回值  pointCut里面写代码不会被执行
         */
        @Pointcut("execution(* top.oitm.practice.aop.*.*(..))")
        public void pointCut(){
        }
        
        @Before("pointCut()")
        public void before(){
            System.out.println("before");
        }                
    }
    
    @Configuration
    @EnableAspectJAutoProxy
    @ComponentScan("top.oitm.practice.aop")
    public class AppConfig {
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
            IndexDao bean = context.getBean(IndexDao.class);
            bean.query();
        }
    }    
    ```