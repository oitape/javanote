- static
    - 全局静态，注意并发读写问题    
    - 只能修饰方法，变量，方法块。静态变量，静态方法块和类无关
        - 如果修饰变量ArrayList有线程安全问题，可替换为`Lists.newCopyOnWriteArrayList();` 或者每次访问手动加锁
        - 修饰方法，只能访问static修饰的变量、方法，不能访问普通方法，static方法内部的变量在执行时没有线程安全问题，方法执行时，数据运行在栈里面，栈的数据每个线程是隔离的
        - 修饰方法块，用于类启动初始化内容
    - 初始化时机
        - 父类静态变量初始化 
        - 父类静态块初始化 
        - 子类静态变量初始化 
        - 子类静态块初始化 
        - main 方法执行 
        - 父类构造器初始化 
        - 子类构造器初始化

- final
    - 修饰类，该类无法继承
    - 修饰方法，该方法无法重写
    - 修饰变量，说明该变量在声明时，就必须初始化完成，而且以后也不能修改其内存地址
    
- try catch finally
    - 代码的执行顺序为:try -> catch -> finally
    - catch 发生了异常，finally 还会执行的，并且是 finally 执行完成之后，才会抛出 catch 中的异常
    
- volatile
    - 内存会主动通知 CPU 缓存。当前共享变量的值已经失效了，你需要重新 来拉取一份，CPU 缓存就会重新从内存中拿取一份最新的值。
    - 加了 volatile 关键字的变量，就会被识别成共享变量，内存 中值被修改后，会通知到各个 CPU 缓存，使 CPU 缓存中的值也对应被修改，从而保证线程从 CPU 缓存中拿取出来的值是最新的。

- transient 
    - transient 关键字我们常用来修饰类变量，意思是当前变量是无需进行序列化的。在序列化时， 就会忽略该变量，这些在序列化工具底层，就已经对 transient 进行了支持
- default
    - default 关键字一般会用在接口的方法上，意思是对于该接口，子类是无需强制实现的，但自己 必须有默认实现
    ```java
    public interface Person {
        default void name(){
            System.out.println("oitm");
        }
    }
    public class Oitm implements Person {}    
    ```