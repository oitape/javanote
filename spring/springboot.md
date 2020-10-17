- springboot启动流程
    - spring ApplicationEvent ——> java事件模型 ——> 观察者设计模式
- @Autowired的装配实现
    - AutowiredAnnotationBeanPostProcessor#postProcessPropertyValues方法中，首先现根据类型进行找，如果找到两个，那么会继续根据属性名字找，找到就不注入进去，存在多个根据名字如果没找到就会抛异常。
    
- Condition
    - @ConditionalOnBean（仅仅在当前spring中存在某个对象时，才会实例化一个Bean）
    - @ConditionalOnClass（某个class位于类路径上，才会实例化一个Bean）

    - @ConditionalOnExpression（当表达式为true的时候，才会实例化一个Bean）
    - @ConditionalOnMissingBean（仅仅在当前上下文中不存在某个对象时，才会实例化一个Bean）
    - @ConditionalOnMissingClass（某个class类路径上不存在的时候，才会实例化一个Bean）
    - @ConditionalOnNotWebApplication（不是web应用）
    - 自定义：@Condition指定自定义的类实现Condition接口

- 代码配置port
```java
    @Bean
    public TomcatServletWebServerFactory tomcatServletWebServerFactory(){
        TomcatServletWebServerFactory serverFactory = new TomcatServletWebServerFactory();
        serverFactory.setPort(9000);
        return serverFactory;
    }
```