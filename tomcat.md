- 启动
    ```sh
    ./bin/catalina.sh start
    ```
    监听socket——>接受请求——>处理请求——>响应请求
- 停止
    ```sh
    telnet 127.0.0.1 8005
    SHUTDOWN
    ```
- Tomcat
    - 一个Server包含多个Connector（链接器）和Container（容器）    
        - Connector：开启socket监听请求响应请求
        - Container：负责具体的请求处理
    - Tomcat流程图
    ![](/assets/iShot2020-09-27上午07.26.54.png)
    - Tomcat生命周期接口静态结构图
    ![](/assets/iShot2020-09-24下午09.55.08.png)
    - Tomcat启动过程时序图
    ![](/assets/Tomcat启动过程时序图.png)
    - Tomcat类加载器结构图
    ![](/assets/iShot2020-09-24下午09.53.46.png)