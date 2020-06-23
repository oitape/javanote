    - 单例模式
    - 饿汉式:在加载的时候就已经被实例化，只有一次 线程安全，
    ```java
    public class HungerySingleton {
        //加载的时候就产生的实例对象,ClassLoader
        private static HungerySingleton instance = new HungerySingleton();
        
        private HungerySingleton() { }
        
        public static HungerySingleton getInstance() {
            return instance;
        }    
    }
    ``` 
    - 懒汉式
    ```java
    public class HoonSynSingleton {
        private static HoonSynSingleton instance = null;
        
        private HoonSynSingleton() {
        }
        
        public static HoonSynSingletonDemo getInstance() {
            if (instance != null) {
                return instance
            }
            synchronized (HoonSynSingletonDemo.class) {
                if (instance != null) {
                    return instance
                }
                return new HoonSynSingletonDemo();
            }
        }                            
    }
    ```    
    - 内部类
    ```java
    public class HolderDemo {
        private HolderDemo() { }
    
        //内部类在什么时候初始化  需要外部类调用内部类才会初始化
        private static class Holder {
            private static HolderDemo instance = new HolderDemo();
        }
    
        public static HolderDemo getInstance() {
            return Holder.instance;
        }
    }

    ```
    - 枚举
    ```java
    
    ```