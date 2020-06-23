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
    
        public synchronized static HoonSynSingleton getInstance() {
            if (null == instance)
                instance = new HoonSynSingleton();
            return instance;
        }                        
    }
    ```