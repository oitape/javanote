- mvc项目可以不用web.xml也可以配置好项目启动
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
    - 新建文件名为`javax.servlet.ServletContainerInitializer`并把类名`top.oitm.Mvcinitializer`写到文件当中。这时候Tomcat在启动的时候就会自动执行Mvcinitializer中的onStartup方法

        
- SpringMVC 核心流程
    - 首先请求进入DispatcherServlet 由DispatcherServlet 从HandlerMappings中提取对应的Handler
    - 此时只是获取到了对应的Handle，然后得去寻找对应的适配器，即:HandlerAdapter
    - 拿到对应HandlerAdapter时，这时候开始调用对应的Handler处理业务逻辑了(这时候实际上已经执行完了我们的 Controller) 执行完成之后返回一个ModeAndView
    - 这时候交给我们的ViewResolver通过视图名称查找出对应的视图然后返回
    - 最后 渲染视图 返回渲染后的视图 --> 响应请求
    
