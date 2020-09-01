- getFields()和getDeclaredFields()区别
    - getFields()：获得某个类的所有的公共（public）的字段，包括父类中的字段。 
    - getDeclaredFields()：获得某个类的所有声明的字段，即包括public、private和proteced，但是不包括父类的申明字段。
    
- setAccessible（true）
    - 将此对象的 accessible 标志设置为指示的布尔值。值为 true 则指示反射的对象在使用时应该取消 Java 语言访问检查。值为 false 则指示反射的对象应该实施 Java 语言访问检查;实际上setAccessible是启用和禁用访问安全检查的开关,并不是为true就能访问为false就不能访问 。
    - Accessible并不是标识方法能否访问的. public的方法 Accessible仍为false；使用了method.setAccessible(true)后 性能有了20倍的提升；Accessable属性是继承自AccessibleObject 类. 功能是启用或禁用安全检查 ；
    
- BeanFactory和FactoryBean的区别
    - BeanFactory：工厂获取spring管理的对象，getBean()
    - FactoryBean：如果类实现了FactoryBean接口，那么spring容器当中存在两个对象。
        - 一个是实现FactoryBean接口getObject()返回的对象, 如果获取，使用BeanFactory的getBean("beanName")
        - 还有一个是当前对象，如果需要获取，使用BeanFactory的getBean("&beanName"),多一个&符号
