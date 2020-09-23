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
    - 简单设计
        - 通常情况下使用Socket监听服务器指定端口来实现该功能，Start()：启动服务器，打开socket连接，监听服务端口，接受客户端请求、处理、返回响应
    Stop()：关闭服务器，释放资源。
        - 缺点：请求监听和请求处理放一起扩展性很差（协议的切换 tomcat独立部署使用HTTP协议，与Apache集成时使用AJP协议）
        - 改进：网络协议与请求处理分离
    - 一个Server包含多个Connector（链接器）和Container（容器）    
        - Connector：开启socket监听请求响应请求
        - Container：负责具体的请求处理
