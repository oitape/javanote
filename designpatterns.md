- 观察者模式
    - 监听者
    ```java
    public class JDKObserver implements Observer {
        @Override
        public void update(Observable o, Object arg) {
            System.out.println("update: arg = " + arg);
            System.out.println("update: JDKObservable = " + o);    
        }
    }
    ```
    - 被监听者
    ```java
    public class JDKObservable extends Observable {
        public void move(){
            // 修改
            setChanged();
            // 通知监听者
            notifyObservers();
            setChanged();
            notifyObservers("args");
        }        
    }
    ```
    - 示例
    ```java
    public static void main(String[] args) {
        JDKObservable movieJDK = new JDKObservable();
        JDKObserver masterJDK = new JDKObserver();
        movieJDK.addObserver(masterJDK);
        movieJDK.move();
    }
    ```