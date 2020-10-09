- Socket构造函数比较多，分两大类
    - 指定代理类型Proxy创建套接字，一共三种类型：DIRECT(直连)、 HTTP(HTTP、FTP 高级协议的代理)、SOCKS(SOCKS 代理)，三种不同的代码方式对 应的 SocketImpl 不同，分别是:PlainSocketImpl、HttpConnectSocketImpl、 SocksSocketImpl，除了类型之外 Proxy 还指定了地址和端口
    - 默认SocksSocketImpl创建，并且需要在构造函数中传入地址和端口
    
- 构造方法
    ```java
    public Socket(InetAddress address, int port) throws IOException {
        this(address != null ? new InetSocketAddress(address, port) : null,
             (SocketAddress) null, true);
    }
    
    // this 底层构造器的源码:
    // stream 为 true 时，表示为stream socket 流套接字，使用 TCP 协议，比较稳定可靠，但占用资源 
    // stream 为 false 时，表示为datagram socket 数据报套接字，使用 UDP 协议，不稳定，但占用资 
    private Socket(SocketAddress address, SocketAddress localAddr,boolean stream) throws IOException {
    ...
    }
    ```

- Socket 常用设置参数
    - setTcpNoDelay：<br> 仅仅对 TCP 生效， 主要为了禁止使用 Nagle 算法，true 表示禁止使用，false 表示使用，默认是 false。如果 Nagle 算法开启，算法会自动合并小数据包，等到达到一定大小(MSS)后，才会和服 务端交互，优点是减少了通信次数，缺点是实时响应度会低一些。
    - setSoLinger：<br> 调用 close 方法时，默认是直接返回的，但如果给 SO_LINGER 赋值，就会阻塞 close 方法，在 SO_LINGER 时间内，等待通信双方发送数据，如果时间过了还未结束，将发送TCP RST强制关闭TCP连接
    - setOOBInline：<br> 如果希望接受 TCP urgent data(TCP 紧急数据)的话，可以开启该选项，默认该 选项是关闭的，我们可以通过 Socket#sendUrgentData 方法来发送紧急数据。
    - setSoTimeout: <br> setSoTimeout 方法主要是用来设置 SO_TIMEOUT 属性的。用来设置阻塞操作的超时时间，阻塞操作主要有:
        - ServerSocket.accept() 服务器等待客户端的连接;
        - SocketInputStream.read() 客户端或服务端读取输入超时; 
        - DatagramSocket.receive()。
    - setSendBufferSize：<br> 设置 SO_SNDBUF 属性的, 设置发送端(输出端)的缓冲区的大小，单位是字节。如果值设置太小，很有可能导致网络交互过于频繁,设置太大，那么交互变少，实时性就会变低。
    - setReceiveBufferSize：<br> 设置 SO_RCVBUF属性，表示设置接 收端的缓冲区的大小，单位是字节。
    - setKeepAlive：<br> 设置 SO_KEEPALIVE 属性, 主要是用来探测服务端的套接字是否还 是存活状态，默认设置是 false，不会触发这个功能。
    
- ServerSocket: 作为服务端的套接字，接受客户端套接字传递过来的信息，并把 响应回传给客户端, 和 Socket 一样，底层都是依靠 SocketImpl 的能力，而 SocketImpl 底层能力 的实现基本上都是 native 方法实现的。

- TCP 有自动检测服务端是否存活的机制么?有没有更好的办法?
    - 可以通过 setKeepAlive 方法来激活该功能，如果两小时内，客户端和服务端的 套接字之间没有任何通信，TCP 会自动发送 keepalive 探测给服务端。并不建议使用这种方式，我们可以自己起一个定时任务，定时的访问服务端的特殊接口，如果服务端返回的数据和预期一致，说明服务端是存活的。
