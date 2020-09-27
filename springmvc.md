- mvc可以不用web.xml也可以配置好项目启动
    - 新建类实现ServletContainerInitializer的onStartup方法
    ```java
    public class Mvcinitializer implements ServletContainerInitializer {
            @Override
            public void onStartup(Set<Class<?>> c, ServletContext servletContext) throws ServletException {
    //        ServletRegistration.Dynamic registration=ctx.addServlet("xx",new SpringServlet());
    //        registration.addMapping("/");
    
            AnnotationConfigWebApplicationContext context = new AnnotationConfigWebApplicationContext();                
            context.register(Appconfig.class);
            context.refresh();
            DispatcherServlet dispatcherServlet = new DispatcherServlet();
            ServletRegistration.Dynamic registration = servletContext.addServlet("xx", dispatcherServlet);
            registration.addMapping("/");
            registration.setLoadOnStartup(1);
            }
    }
    ```
    
  - 新建resources目录
    - 在META-INF目录建services目录
    - 新建文件