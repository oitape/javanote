- 单例模式
    - 饿汉式
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