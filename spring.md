- spring可以把相同类型注入到map中
   ```java
   // key是类名首字母小写
   @Autowire
   private Map<String,OrderDao> daos;
   ```
- 在setter方法中添加@Required注解，该类实例化的时候必须设置该属性

- spring默认的不是自动装配：AUTOWIRE_NO
    - 为属性添加@Autowire才会注入进来
    - 比如mybatis是在MapperFactoryBean中通过ClasPathMapperScanner修改了beanDefinition的autowire类型为AUTOWIRE_BY_TYPE
    
- Spring源码流程
```
//实例化一个工厂DefaultListableBeanFactory
org.springframework.context.support.GenericApplicationContext->GenericApplicationContext()
  	1、实力化一个AnnotatedBeanDefinitionReader
	2、ClassPathBeanDefinitionScanner，能够扫描我们bd,能够扫描一个类，并且转换成bd
	org.springframework.context.annotation.AnnotationConfigApplicationContext#AnnotationConfigApplicationContext()
		委托AnnotationConfigUtils
		org.springframework.context.annotation.AnnotatedBeanDefinitionReader#AnnotatedBeanDefinitionReader()
			
			org.springframework.context.annotation.AnnotationConfigUtils#registerAnnotationConfigProcessors()
			
				1、添加AnnotationAwareOrderComparator类的对象，主要去排序
				2、ContextAnnotationAutowireCandidateResolver
				3、往BeanDefinitionMap注册一个ConfigurationClassPostProcessor?  org.springframework.context.annotation.internalConfigurationAnnotationProcessor
					why?因为需要在invokeBeanFactoryPostProcessors
					invokeBeanFactoryPostProcessors主要是在spring的beanFactory初始化的过程中去做一些事情，怎么来做这些事情呢？
					委托了多个实现了BeanDefinitionRegistryPostProcessor或者BeanFactoryProcessor接口的类来做这些事情,有自定义的也有spring内部的
					其中ConfigurationClassPostProcessor就是一个spring内部的BeanDefinitionRegistryPostProcessor
					因为如果你不添加这里就没有办法委托ConfigurationClassPostProcessor做一些功能
					到底哪些功能？参考下面的注释
				4、RequiredAnnotationBeanPostProcessor
				.......
				org.springframework.context.annotation.AnnotationConfigUtils#registerAnnotationConfigProcessors()
					//往BeanDefinitionMap注册
					org.springframework.context.annotation.AnnotationConfigUtils#registerPostProcessor
						//准备好bean工厂，实例化对象
						org.springframework.context.support.AbstractApplicationContext#refresh
						//准备工作包括设置启动时间，是否激活标识位， 初始化属性源(property source)配置
							org.springframework.context.support.AbstractApplicationContext#prepareRefresh
								//得到beanFactory?因为需要对beanFactory进行设置
								org.springframework.context.support.AbstractApplicationContext#obtainFreshBeanFactory
									//准备bean工厂
									1、添加一个类加载器
									2、添加bean表达式解释器，为了能够让我们的beanFactory去解析bean表达式
									3、添加一个后置处理器ApplicationContextAwareProcessor
									4、添加了自动注入别忽略的列表
									5、。。。。。。
									6、添加了一个ApplicationListenerDetector后置处理器（自行百度）
									org.springframework.context.support.AbstractApplicationContext#prepareBeanFactory
										目前没有任何实现
										org.springframework.context.support.AbstractApplicationContext#postProcessBeanFactory
											1、getBeanFactoryPostProcessors()得到自己定义的（就是程序员自己写的，并且没有交给spring管理，就是没有加上@Component）
											2、得到spring内部自己维护的BeanDefinitionRegistryPostProcessor
											org.springframework.context.support.AbstractApplicationContext#invokeBeanFactoryPostProcessors
												//调用这个方法
												//循环所有的BeanDefinitionRegistryPostProcessor
												//该方法内部postProcessor.postProcessBeanDefinitionRegistry
												org.springframework.context.support.PostProcessorRegistrationDelegate#invokeBeanDefinitionRegistryPostProcessors
													//调用扩展方法postProcessBeanDefinitionRegistry
													org.springframework.beans.factory.support.BeanDefinitionRegistryPostProcessor#postProcessBeanDefinitionRegistry
														//拿出的所有bd，然后判断bd时候包含了@Configuration、@Import，@Compent。。。注解
														org.springframework.context.annotation.ConfigurationClassPostProcessor#processConfigBeanDefinitions
															1、的到bd当中描述的类的元数据（类的信息）
															2、判断是不是加了@Configuration   metadata.isAnnotated(Configuration.class.getName())
															3、如果加了@Configuration，添加到一个set当中,把这个set传给下面的方法去解析
															org.springframework.context.annotation.ConfigurationClassUtils#checkConfigurationClassCandidate
															//扫描包																														org.springframework.context.annotation.ConfigurationClassParser#parse(java.util.Set<org.springframework.beans.factory.config.BeanDefinitionHolder>)
																
																org.springframework.context.annotation.ConfigurationClassParser#parse(org.springframework.core.type.AnnotationMetadata, java.lang.String)
																	//就行了一个类型封装
																	org.springframework.context.annotation.ConfigurationClassParser#processConfigurationClass
																	1、处理内部类 一般不会写内部类
																	org.springframework.context.annotation.ConfigurationClassParser#doProcessConfigurationClass
																		//解析扫描的一些基本信息，比如是否过滤，比如是否加入新的包。。。。。
																		org.springframework.context.annotation.ComponentScanAnnotationParser#parse
																			org.springframework.context.annotation.ClassPathBeanDefinitionScanner#doScan
																			org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#findCandidateComponents
																				org.springframework.context.annotation.ClassPathScanningCandidateComponentProvider#scanCandidateComponents
```

- ConfigurationClassPostProcessor
   - processConfigBeanDefinitions
     - 是否有@Configuration注解，spring会加一个标记
     - 加：CONFIGURATION_CLASS_FULL，回调用：enhanceConfigurationClasses，使用cglib动态代理生成代理的config代理类，类中增加了($$beanFactory)成员变量，重复调用类中加了@Bean的方法只会产生一次，不会执行多次对象的生成。内部执行会判断是不是第一次执行，如果是创建，不是就使用已创建的对象。ConfigurationClassEnhancer.isCurrentlyInvokedFactoryMethod
     - 不加：CONFIGURATION_CLASS_LITE，类中加了@Bean的方法，对象的构造方法被调用多少次就执行多少次
    
- 四种类注册
   - 普通类：扫描完成后注册
   - ImportSelector：先configurationClasses；然后再注册
   - Registrar   importBeanDefinitonRegistrars 再注册
   - Import普通类： 先configurationClasses
   

- BeanPostProcessor: 插手bean的实例化过程，在bean实例化之后，bean还没有被spring的bean容器管理之前干预。使用场景@PostConstruct、@Aspectj
- BeanFactoryProcessor：任意一个bean被new出来之前执行。场景：ConfigurationClassPostProcessor#postProcessBeanFactory，用来针对配置类加上cglib动态代理
- BeanDefinitionRegistryPostProcessor：是BeanPostProcessor的子类在BeanFactoryProcessor之前执行：应为源码当中先遍历BeanDefinitionRegistryPostProcessor（有自定义的还有spring提供的），自定义的先执行，ConfigurationClassPostProcessor扫描、3种import的扫描、@Bean的扫描，判断配置类是否是一个完整的配置类
- importSelector：通过这个方法selectImports返回一个全类名，把他变成beanDefinition，动态添加bd，这个bd是死的，不能动态变化。
- ImportBeanDefinitionRegistrar: 可以动态注册，可实现的功能包含importSelector，场景：mybatis

- spring AbstractBeanFactory.doGetBean中使用transformedBeanName的原因？
![](/assets/iShot2020-09-23上午06.13.19.png)
	- 每次doGetBean都会判断，该name的bean是否已创建
	```java
	// Eagerly check singleton cache for manually registered singletons.
		Object sharedInstance = getSingleton(beanName);
	```
	- 和下面的判断实现不一样,要区分开，该方法内部会进行bean的创建
	```java
	// Create bean instance.
	if (mbd.isSingleton()) {
		sharedInstance = getSingleton(beanName, () -> {
			try {
				return createBean(beanName, mbd, args);
			}
			catch (BeansException ex) {
				// Explicitly remove instance from singleton cache: It might have been put there
				// eagerly by the creation process, to allow for circular reference resolution.
				// Also remove any beans that received a temporary reference to the bean.
				destroySingleton(beanName);
				throw ex;
			}
		});
		bean = getObjectForBeanInstance(sharedInstance, name, beanName, mbd);
	}
	```	
	- 最后创建的实现在`AbstractAutowireCapableBeanFactory.doCreateBean`