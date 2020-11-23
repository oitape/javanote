- RMQ整合Spring使用
    - 依赖
    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-amqp</artifactId>
    </dependency>
    
    spring:
          rabbitmq:
            host:
            port:
            username:
            password:
            virtual-host: 

    ```
    - 生产者
    ```java
    @Bean
    public ConnectionFactory connectionFactory() {
        CachingConnectionFactory connectionFactory = new CachingConnectionFactory("localhost",5672);
        //在构造方法传入了
        //        connectionFactory.setHost();
        //        connectionFactory.setPort();
        connectionFactory.setUsername("admin");
        connectionFactory.setPassword("admin");
        connectionFactory.setVirtualHost("testhost");
        //是否开启消息确认机制
        //connectionFactory.setPublisherConfirms(true);
        return connectionFactory;
    }
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) {
        //注意  这个ConnectionFactory 是使用javaconfig方式配置连接的时候才需要传入的  如果是yml配置的连接的话是不需要的
        RabbitTemplate template = new RabbitTemplate(connectionFactory)；
        return template;
    }
    
    @Bean
    public DirectExchange  defaultExchange() {
        return new DirectExchange("directExchange");
    }

    @Bean
    public Queue queue() {
        //名字  是否持久化
        return new Queue("testQueue", true);
    }

    @Bean
    public Binding binding() {
        //绑定一个队列  to: 绑定到哪个交换机上面 with：绑定的路由建    （routingKey）
        return                 BindingBuilder.bind(queue()).to(defaultExchange()).with("direct.key");
    }
    
    @Component
    public class TestSend {
    
        @Autowired
        RabbitTemplate rabbitTemplate;
        public void send() {
            //至于为什么调用这个API 后面会解释
            //参数介绍： 交换机名字，路由建， 消息内容
            rabbitTemplate.convertAndSend("directExchange", "direct.key", "hello");
        }
    }
    ```
    - 消费者
    ```java
    @Component
    public class TestListener  {
        @RabbitListener(queues = "testQueue")
        public void get(String message) throws Exception{
            System.out.println(message);
        }
    }
    ```