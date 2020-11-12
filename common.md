
- jdk和jre的区别
    - jre：提供java的运行环境
    - jdk：提供java的开发环境和运行环境（包含了jre）
- 基本数据类型及其封装类
    ![](/assets/iShot2020-10-12下午06.30.51.png)
- 为什么需要封装类？
    - 因为泛型类包括预定义的集合，使用的参数都是对象类型，无法直接使用基本数据类型
    - 基本数据类型按值传递，封装类型按引用传递
    - 基本类型在栈中创建，封装类型在堆中创建
- ==和equals区别？
    - ==比较的是引用而equals比较的是内容
- Java没有全局变量
    - 全局变量破坏了引用透明性原则。全局变量导致命名空间冲突
- do while和while区别
    - do的判断是在尾部，至少会执行一次循环体
- float f=3.4；是否正确
    - 不争取，3.4是双精度数，将双精度复制给浮点型属于下转型会造成精度损失，因此需要强制类型转换**float f=3.4F** 
- final、finally、finalize区别？
    - final
        - final修饰的类无法被继承
        - 修饰属性，基础类型一旦赋值无法改变，引用类型在对其初始化之后便不能让其指向另一个对象，但是指向的对象内容可变的
        - final修饰的方法无法被重写，但允许重载
        - 类的private方法会隐式的被指定为final方法
    - finally
        - 异常最终清理相关资源，最终操作
        - 一般情况下会被执行。极端情况不会，如果对应try块没有执行，则这个try块的finally也不会执行；如果try块执行时jvm关机了也不会执行。
    - finalize
        - 是Object类的protected方法，子类可以覆盖该方法以实现资源清理工作
        - GC在回收对象前会调用该方法
        - 存在的问题？
            - java语言规范并不保证该方法会被及时执行，更不保证他们一定执行
            - 可能带来性能问题，因为JVM通常在单独的低优先级线程执行该方法
            - 最多有GC执行一次，可手动调用
- hashCode和equals区别？
    - 重写equals一般比较全面比较复杂，这样效率较低，hash只需要生成一个值就可以比较，效率高
    - hash虽然效率高但不够可靠，不同的对象可产生相同的hash
    - 只要重写equals 最好必须重写下hashCode
    
- 四种引用
    - 强引用：`String s = "abc"`就是强引用，只要强引用在，则垃圾回收就不会回收这个对象
    - 软引用：用于描述还有用但非必须的对象，如果内存足够，不回收，内存不足，则回收
    - 弱引用：只具有弱引用的对象拥有更短暂的生命周期，在垃圾回收器线程扫描它所管辖的内存区域个过程中，一旦发现只具有弱引用的对象，不管当前内存空间是否足够，都会回收
    - 虚引用：虚引用不会决定对象的生命周期，如果一个对象仅持有虚引用，那么和没有任何引用一样
    
- 泛型中extends和super区别
    - Java泛型中的“通配符（Wildcards）”和“边界（Bounds）”的概念。
    - extends：是指 “上界通配符（Upper Bounds Wildcards）”
    - super：是指 “下界通配符（Lower Bounds Wildcards）”
    - 多接口限定：`T extends SomeClass & interface1 & interface2 & interface3`
        ```java
    class Test <T extends Color & Brand> {
            public static <T extends Color & Brand> void test(T t) {
            t.getBrand();
    }    
            public void testCar(T t){
            t.getColor();
    }    
        }
        class Car implements Color, Brand {}
        ```

    
    
    
    
    