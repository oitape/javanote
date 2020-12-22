## 基本概念
- Java 源文件，通过编译器，能够生产相应的.Class 文件，也就是字节码文件， 而字节码文件又通过 Java 虚拟机中的解释器，编译成特定机器上的机器码。
- 不只是专用于Java语言，只要生成的编译文件匹配JVM对加载编译文件格式要求，任何语言都可以由JVM编译运行。 比如kotlin、scala等。
    
### JVM基本结构
- 类加载子系统
- 运行时数据区（内存结构）
- 执行引擎
![](/assets/jvm.png)

### 运行时数据区
- 方法区(Method Area)
    - 类的所有字段和方法字节码，以及一些特殊方法如构造函数，接口代码也在这里定义。所有定义的方法的信息都保存在该区域，静态变量+常量+类信息(构造方法/接口定义)+运行时常量池都存在方法区中，虽然Java虚拟 机规范把方法区描述为堆的一个逻辑部分，但是它却有一个别名叫做Non-Heap(非堆)，目的应该是为了和Java的 堆区分开

- 堆(Heap)			
    - 使用new创建的对象，定义的数组都存在堆区。垃圾回收机制算法、jvm参数调优、内存溢出和内存泄漏等都是在堆内存中进行操作。
    - 堆内存划分：新生代和老生代，
        - 新生代伊甸区(Eden space)和幸存者区(Survivor space)，所有的类都是在伊甸区被new出来的，幸存区又分为From和To区，当Eden区的空间用完是，程序又需要创建对象，JVM的垃圾回收器将Eden区进行垃圾回 收(Minor GC)，将Eden区中的不再被其它对象应用的对象进行销毁。然后将Eden区中剩余的对象移到From Survivor区。若From Survivor区也满了，再对该区进行垃圾回收，然后移动到To Survivor区。
        - Eden:from:to = 8:1:1
        - 新老默认内存比例1：2。
        - 新生代：刚出生不久的对象，存放在新生代里面；存放不是经常使用的对象。
        - 老生代：新生代经过多次GC仍然存货的对象移动到老年区。若老年代也满了，这时候将发生Major GC(也可以叫Full GC)， 进行老年区的内存清理。若老年区执行了Full GC之后发现依然无法进行对象的保存，就会抛出OOM(OutOfMemoryError)异常，存放比较活跃的对象；存放经常被引用对象。
    - 元空间(Meta Space)
        - JDK1.8之后，元空间替代了永久代，它是对JVM规范中方法区的实现，区别在于元数据区不在虚拟机当中，而是用的本地内存，永久代在虚拟机当中，永久代逻辑结构上也属于堆，但是物理上不属于。

### 栈(Stack)
- Java线程执行方法的内存模型，一个线程对应一个栈，每个方法在执行的同时都会创建一个栈帧(用于存储局部变量 表，操作数栈，动态链接，方法出口等信息)不存在垃圾回收问题，只要线程一结束该栈就释放，生命周期和线程一致

### 本地方法栈(Native Method Stack) 
- 和栈作用很相似，区别不过是Java栈为JVM执行Java方法服务，而本地方法栈为JVM执行native方法服务。登记native方法，在Execution Engine执行时加载本地方法库

### 程序计数器(Program Counter Register)
- 就是一个指针，指向方法区中的方法字节码(用来存储指向吓一跳指令的地址，也即将要执行的指令代码)，由执行 引擎读取下一条指令，是一个非常小的内存空间，几乎可以忽略不计

### 垃圾回收机制
- JVM不定时的回收不可达的对象（对象未被引用）
```java
///提示给gc进行回收垃圾，但不代表立即进行回收。gc线程是守护线程
	System.gc();
	
	///垃圾回收之前会执行的方法
	protected void finalize() throws Throwable {
		super.finalize();
}
```
### 双亲委派机制
- JVM提供三层ClassLoader
    - Bootstrap classLoader:主要负责加载核心的类库(java.lang.*等)，构造ExtClassLoader和APPClassLoader。
    - ExtClassLoader：主要负责加载jre/lib/ext目录下的一些扩展的jar。
    - AppClassLoader：主要负责加载应用程序的主函数类
    
- 先委托父类加载器寻找目标类，在找不到的情况下，再加载自己的路径中查找目标类
    - 沙箱安全机制:比如自己写的String.class类不会被加载，这样可以防止核心库被随意篡改 
    - 避免类的重复加载:当父ClassLoader已经加载了该类的时候，就不需要子ClassLoader再加载一次

### JVM性能调优工具
- Jinfo
    - Jinfo -flags 7888：查看JVM的参数
    - Jinfo -sysProps  7888: 查看java系统属性
- Jstat：可以查看堆内存各部分使用量，以及加载类的数量
    - Jstat -class 7888：类加载统计
    - Jstat -gc 7888
- Jmap：查看内存信息
    - jmap -histo 7824 > xxx.txt：堆的对象统计

### 内存溢出和内存泄漏的区别：
- Java内存泄漏就是没有及时清理内存垃圾，导致系统无法再给你提供内存资源（内存资源耗尽）；而Java内存溢出就是你要求分配的内存超出了系统能给你的，系统不能满足需求，于是产生溢出。
- 内存溢出，这个好理解，说明存储空间不够大。就像倒水倒多了，从杯子上面溢出了来了一样。
- 内存泄漏，原理是，使用过的内存空间没有被及时释放，长时间占用内存，最终导致内存空间不足，而出现内存溢出。

### G1垃圾回收
- 并行与并发:G1能充分利用CPU、多核环境下的硬件优势，使用多个CPU(CPU或者CPU核心)来缩短Stop- The-World停顿时间。部分其他收集器原本需要停顿Java线程执行的GC动作，G1收集器仍然可以通过并发的方 式让java程序继续执行 
- 分代收集:虽然G1可以不需要其他收集器配合就能独立管理整个GC堆，但是还是保留了分代的概念。 空间整 合:与CMS的“标记–清理”算法不同，G1从整体来看是基于“标记整理”算法实现的收集器;从局部上来看是基 于“复制”算法实现的
- 可预测的停顿:这是G1相对于CMS的另一个大优势，降低停顿时间是G1 和 CMS 共同的关注点，但G1 除了追 求低停顿外，还能建立可预测的停顿时间模型，能让使用者明确指定在一个长度为M毫秒的时间片段内

大致步骤：
    - 初始标记
    - 并发标记
    - 最终标记  
    - 筛选回收

G1收集器在后台维护了一个优先列表，每次根据允许的收集时间，优先选择回收价值最大的Region(这也就是它的名 字Garbage-First的由来)。这种使用Region划分内存空间以及有优先级的区域回收方式，保证了GF收集器在有限时间 内可以尽可能高的收集效率(把内存化整为零)

- 配置G1收集器
```
-XX:+UseG1GC
```
- 第一次调优，设置Metaspace大小:增大元空间大小-XX:MetaspaceSize=64M -XX:MaxMetaspaceSize=64M 
- 第二次调优，添加吞吐量和停顿时间参数:-XX:GCTimeRatio=99 -XX:MaxGCPauseMillis=10

### GC调优步骤
- 打印GC日志
```
-XX:+PrintGCDetails -XX:+PrintGCTimeStamps -XX:+PrintGCDateStamps -Xloggc:./gc.log
```
- 分析日志得到关键性指标
- 分析GC原因，调优JVM参数


### GC常用参数
#### 堆栈设置
    - -Xss:每个线程的栈大小 
    - -Xms:初始堆大小，默认物理内存的1/64 
    - -Xmx:最大堆大小，默认物理内存的1/4 
    - -Xmn:新生代大小
    - -XX:NewSize:设置新生代初始大小 
    - -XX:NewRatio:默认2表示新生代占年老代的1/2，占整个堆内存的1/3。 
    - -XX:SurvivorRatio:默认8表示一个survivor区占用1/8的Eden内存，即1/10的新生代内存。 
    - -XX:MetaspaceSize:设置元空间大小 
    - -XX:MaxMetaspaceSize:设置元空间最大允许大小，默认不受限制，JVM Metaspace会进行动态扩展。

#### 垃圾回收统计信息
- -XX:+PrintGC 
- -XX:+PrintGCDetails 
- -XX:+PrintGCTimeStamps 
- -Xloggc:filename


#### 收集器设置
- -XX:+UseSerialGC:设置串行收集器 
- -XX:+UseParallelGC:设置并行收集器 
- -XX:+UseParallelOldGC:老年代使用并行回收收集器 
- -XX:+UseParNewGC:在新生代使用并行收集器 
- -XX:+UseParalledlOldGC:设置并行老年代收集器 
- -XX:+UseConcMarkSweepGC:设置CMS并发收集器 
- -XX:+UseG1GC:设置G1收集器 
- -XX:ParallelGCThreads:设置用于垃圾回收的线程数


#### G1收集器设置
- -XX:+UseG1GC:使用G1收集器
- -XX:ParallelGCThreads:指定GC工作的线程数量 
- -XX:G1HeapRegionSize:指定分区大小(1MB~32MB，且必须是2的幂)，默认将整堆划分为2048个分区 
- -XX:GCTimeRatio:吞吐量大小，0-100的整数(默认9)，值为n则系统将花费不超过1/(1+n)的时间用于垃圾收集 
- -XX:MaxGCPauseMillis:目标暂停时间(默认200ms) 
- -XX:G1NewSizePercent:新生代内存初始空间(默认整堆5%) 
- -XX:G1MaxNewSizePercent:新生代内存最大空间
- -XX:TargetSurvivorRatio:Survivor填充容量(默认50%)
- -XX:MaxTenuringThreshold:最大任期阈值(默认15) 
- -XX:InitiatingHeapOccupancyPercen:老年代占用空间超过整堆比IHOP阈值(默认45%),超过则执行混合收集 
- -XX:G1HeapWastePercent:堆废物百分比(默认5%) 
- -XX:G1MixedGCCountTarget:参数混合周期的最大总次数(默认8)

### 结论
- 初始堆值和最大堆内存越大，吞吐量就越高。
- 垃圾回收次数和设置最大堆内存大小无关，只和初始内存有关系。初始内存会影响吞吐量。
- 最好使用并行收集器, 因为并行收集器速度比串行吞吐量高，速度快。
- 设置堆内存新生代的比例和老年代的比例最好为1:2或者1:3。
- 减少GC对老年代的回收。


