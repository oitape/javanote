- cpu一致性问题解决
    - 总线加锁（粒度太大）
    - MESI协议
        - 读操作，不做任何事情，把cache中数据读到寄存器
        - 写操作，发出信号通知其他的CPU将该变量的Cache Line置无效，其他CPU要访问这个变量的时候，只能从内存中获取。![](/assets/cpu_sync.png)
    
- Volatile
    作用： 让其他线程马上感知到变量的修改
    - 保证可见性：对共享变量的修改，其他线程马上能感知到，不能保证原子性
    - 保证有序性
    - 规则：volatile修饰的变量
        - volatile之前的代码不能调整到他后面
        - volatile之后的代码不能调整到他前面（as if seria）
        - 霸道（位置不变化）

- Volatile实现原理：
    - 锁、轻量级
    - HSDIS工具：反编译 汇编。将Java——>class——>JVM——>ASM文件
    - Volatile int a; 会看到有 lock :a

- Volatile使用场景
    - 状态标志（开关模式）
    ```java
    public class ShutDowsnDemmo extends Thread{
        private volatile boolean started=false;
        @Override
        public void run() {
            while(started){
                //dowork();
            }
        }
        public void shutdown(){
            started=false;
        }                    
    }
    ```
    
    - 双重检查锁定

    ```java
    public class Singleton {
        private volatile static Singleton instance;
        public static Singleton getInstance(){
            if(instance==null){    
                synchronized (Singleton.class){
                    instance=new Singleton();
                }
            }
        return instance;
        }
    }    
    ```
    
- Volatile与Synchronized
    - 使用上的区别：
        Volatile只能修饰变量，Synchronized只能修饰方法和语句块
    - 原子性保证：
        Volatile不保证原子性，Synchronized保证原子性
    - 可见性：
        都可以保证可见性，但实现原理不同
        `Volatile对变量加了lock，synchronized使用monitorEnter和monitorexit  monitor  JVM`
    - 有序性
        Volatile保证有序性，synchronized也可以保证但代价高 类似于串行
    - 阻塞
        Volatile不会引起阻塞，synchronized引起阻塞
    
    
    
    
    