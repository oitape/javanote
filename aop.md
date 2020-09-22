- 使用
    ```java
    @Component
    @Aspect
    public class AspectJ {
        /**
         * execution可以定义到方法、入参类型、返回值
         * aop包所有类 所有方法 任意入参(如果要指定入参类型写完整类路径) 任意返回值 任意方法类型
         * pointCut里面写代码不会被执行
         */
        @Pointcut("execution(* top.oitm.practice.aop.*.*(..))")
        public void pointCut(){
        }
        
        @Before("pointCut()")
        public void before(){
            System.out.println("before");
        }  
        
        //第二种
        /**
         * within只能定义到里类
         @Pointcut("within(top.oitm.practice.aop.*)")
         public void within(){
         }
         *args 指定参数
         @Pointcut("args(java.lang.String,java.lang.Integer)")
         public void args(){
        
         }
         *this 代表的是当前生成的代理对象
         @Pointcut("this(top.oitm.dao.IndexDao)")
         public void args(){
         } 
    
         *可以指定多个条件PointCut点
         @Before("withIn()&&!args")
         public void before(){
         System.out.println("before");
         }
         */
    }
    ```
    
    ```java
    @Configuration
    //使用Java @Configuration启用@AspectJ支持，请添加@EnableAspectJAutoProxy注释
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
- JDK的动态代理只能基于接口，为什么不能用继承？ 
    - 因为java是单继承的，JDK动态代理对象自动继承了Proxy类，只能通过实现指定的接口类做代理功能。
    -案例
    ```java
    @Component
    @Aspect
    public class AspectJ {
        //this是代理对象，target是原对象
        @Pointcut("this(top.oitm.practice.aop.IndexDao)")
        public void pointThis(){}
    
        @Before("pointThis()")
        public void before(){
            System.out.println("before");
        }
    }

    @Configuration
    @EnableAspectJAutoProxy
    @ComponentScan("top.oitm.practice.aop")
    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext(AppConfig.class);
        //下面的用法会报错：No qualifying bean of type 'top.oitm.practice.aop.IndexDao' available
        Dao bean = (Dao) context.getBean("indexDao");
        bean.query();
    }
    ```
    - 可以通过@EnableAspectJAutoProxy(proxyTargetClass = true)设定为cglib动态代理直接代理指定对象 通过父子类实现
    - 或者使用@Pointcut("target(top.oitm.practice.aop.IndexDao)") 也可以
    


- 字符串变为java class文件，动态为各种目标对象增加代理功能

```java
public class ProxyUtil {
    /**
     * content --->string
     * .java  io
     * .class
     * .new   反射----》class
     * @return
     */
    public static Object newInstance(Object target) {
        Object proxy = null;
        Class targetInf = target.getClass().getInterfaces()[0];
        Method methods[] = targetInf.getDeclaredMethods();
        String line = "\n";
        String tab = "\t";
        String infName = targetInf.getSimpleName();
        String content = "";
        String packageContent = "package com.google;" + line;
        String importContent = "import " + targetInf.getName() + ";" + line;
        String clazzFirstLineContent = "public class $Proxy implements " + infName + "{" + line;
        String filedContent = tab + "private " + infName + " target;" + line;
        String constructorContent = tab + "public $Proxy (" + infName + " target){" + line
                + tab + tab + "this.target =target;"
                + line + tab + "}" + line;
        String methodContent = "";
        for (Method method : methods) {
            String returnTypeName = method.getReturnType().getSimpleName();
            String methodName = method.getName();
            // Sting.class String.class
            Class args[] = method.getParameterTypes();
            String argsContent = "";
            String paramsContent = "";
            int flag = 0;
            for (Class arg : args) {
                String temp = arg.getSimpleName();
                //String
                //String p0,Sting p1,
                argsContent += temp + " p" + flag + ",";
                paramsContent += "p" + flag + ",";
                flag++;
            }
            if (argsContent.length() > 0) {
                argsContent = argsContent.substring(0, argsContent.lastIndexOf(",") - 1);
                paramsContent = paramsContent.substring(0, paramsContent.lastIndexOf(",") - 1);
            }

            methodContent += tab + "public " + returnTypeName + " " + methodName + "(" + argsContent + ") {" + line
                    + tab + tab + "System.out.println(\"log\");" + line
                    + tab + tab + "target." + methodName + "(" + paramsContent + ");" + line
                    + tab + "}" + line;

        }

        content = packageContent + importContent + clazzFirstLineContent + filedContent + constructorContent + methodContent + "}";

        File file = new File("d:\\com\\google\\$Proxy.java");
        try {
            if (!file.exists()) {
                file.createNewFile();
            }

            FileWriter fw = new FileWriter(file);
            fw.write(content);
            fw.flush();
            fw.close();


            JavaCompiler compiler = ToolProvider.getSystemJavaCompiler();

            StandardJavaFileManager fileMgr = compiler.getStandardFileManager(null, null, null);
            Iterable units = fileMgr.getJavaFileObjects(file);

            JavaCompiler.CompilationTask t = compiler.getTask(null, fileMgr, null, null, null, units);
            t.call();
            fileMgr.close();

            URL[] urls = new URL[]{new URL("file:D:\\\\")};
            URLClassLoader urlClassLoader = new URLClassLoader(urls);
            Class clazz = urlClassLoader.loadClass("com.google.$Proxy");

            Constructor constructor = clazz.getConstructor(targetInf);

            proxy = constructor.newInstance(target);
        } catch (Exception e) {
            e.printStackTrace();
        }
        return proxy;
    }
}
```

- spring添加注解@EnableAspectJAutoProxy实现aop的代理
    - @Import(AspectJAutoProxyRegistrar.class)
    - 放到spring的beanDefinitionMap中：AopConfigUtils.registerAspectJAnnotationAutoProxyCreatorIfNecessary(registry);
    ```java
    RootBeanDefinition beanDefinition = new RootBeanDefinition(cls);
    beanDefinition.setSource(source);
    beanDefinition.getPropertyValues().add("order", -2147483648);
    beanDefinition.setRole(2);
    registry.registerBeanDefinition("org.springframework.aop.config.internalAutoProxyCreator", beanDefinition);
    ```
    - registerBeanPostProcessors方法中从beanDefinitionMap中根据类型找出名字
    - 添加到spring后置处理器列表中
    - 把原生对象变成代理对象




