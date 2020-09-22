- JDK动态代理：jdk动态代理，只能对实现了接口的类进行，没有实现接口的类，不能使用JDK动态代理。
利用反射机制生成一个实现
代理接口的匿名类，在调用
具体方法之前调用InvokeHandler
来处理


```java
public static void main(String[] args) {
	IUserDao userDao = new UserDaoImpl();
	InvocationHandlerImpl invocationHandlerImpl = new InvocationHandlerImpl(userDao);
	ClassLoader loader = userDao.getClass().getClassLoader();
	Class<?>[] interfaces = userDao.getClass().getInterfaces();
	//调用动态代理实例
	IUserDao userDao2 = (IUserDao)Proxy.newProxyInstance(loader, interfaces, invocationHandlerImpl);
	userDao2.add();
}


//相当于每次动态生成代理类对象，失信了InvocationHandler接口的调用处理器对象
public class InvocationHandlerImpl implements InvocationHandler {
	private Object target;
	public InvocationHandlerImpl(Object target) {
		this.target = target;
	}
	public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
		System.out.println("开启事务");
		Object invoke = method.invoke(target, args);
		System.out.println("提交事务");
		return invoke;
	}
}

```


- CGLIB动态代理：利用asm开源包，对代理对象类的class字节码文件加载进来通过修改其字节码生成子类处理

```java
public static void main(String[] args) {
	CglibProxy cglibProxy = new CglibProxy();
	UserDaoImpl userDaoImpl = (UserDaoImpl)cglibProxy.getInstance(new UserDaoImpl());
	userDaoImpl.add();
}	

public class CglibProxy implements MethodInterceptor{
	private Object targetObject;
	public Object getInstance(Object target){
		this.targetObject = target;
		//操作字节码 生成虚拟子类
		Enhancer enhancer = new Enhancer();
		enhancer.setSuperclass(target.getClass());
		enhancer.setCallback(this);
		return enhancer.create();
	}
	public Object intercept(Object arg0, Method arg1, Object[] arg2, MethodProxy proxy) throws Throwable {
		System.out.println("开启事务...");
		Object invoke = proxy.invoke(targetObject, arg2);
		System.out.println("提交事务...");
		return invoke;
	}
}

```

- Spring AOP代理策略
	- 如果目标对象实现了接口，默认情况下会采用JDK的动态代理实现AOP 
	- 如果目标对象实现了接口，可以强制使用CGLIB实现AOP 
	- 如果目标对象没有实现了接口，必须采用CGLIB库，spring会自动在JDK动态代理和CGLIB之间转换


- Spring aop不管是使用jdk还是cglib都是动态在运行时植入的，如果用aspectj它是在编译时把类编译成代理的。



