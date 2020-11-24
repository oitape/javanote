### RMQ整合Spring使用
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
    
- 确保消息发送到RMQ
    使用失败回调的方式，也必须开启发送方确认
    - RabbitmqTemplate
    ```java
    @Bean
    public RabbitTemplate rabbitTemplate(ConnectionFactory connectionFactory) { 
        RabbitTemplate template = new RabbitTemplate(connectionFactory); 
        //开启mandatory模式(开启失败回调)
        template.setMandatory(true);
        //指定失败回调接口的实现类 
        template.setReturnCallback(new MyReturnCallback()); 
        return template;
    }    
    ```
    - 回调接口
    ```java
    public class MyReturnCallback implements RabbitTemplate.ReturnCallback {
         @Override
        public void returnedMessage(Message message, int replyCode, String replyText,String exchange, String routingKey){
        }
    }
    ```
    
- 开启发送方确认模式
    ```java
     connectionFactory.setPublisherConfirms(true);
    ```
    - yml
    ```yaml
     spring:
         rabbitmq:
            publisher-confirms: true
    ```
    - 实现接口
    ```java
     public class MyConfirmCallback implements RabbitTemplate.ConfirmCallback{
        @Override
        public void confirm(CorrelationData correlationData, boolean ack, String cause) { 
        } 
    }
    ```
    - RabbitmqTemplate修改
    ` template.setConfirmCallback(new MyConfirmCallback());`
    
- 要注意的是 confirm模式的发送成功 的意思是发送到RabbitMq(Broker)成功 而不是发送到队列成功,所以要和失败回调结合使用,这样才能确认消息投递成功了，
  - confirm机制是确认我们的消息是否投递到了 RabbitMq(Broker)上面，而        
  - mandatory是在我们的消息进入队列失败时候不会被遗弃(让我们自己进行处理)
    
### 消费者确认
- Container配置
```java
@Bean
public SimpleRabbitListenerContainerFactory
simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
        new SimpleRabbitListenerContainerFactory();
    //这个connectionFactory就是我们自己配置的连接工厂直接注入进来             
                                          
    simpleRabbitListenerContainerFactory
        .setConnectionFactory(connectionFactory); 
    //这边设置消息确认方式由自动确认变为手动确认     
    simpleRabbitListenerContainerFactory
        .setAcknowledgeMode(AcknowledgeMode.MANUAL); 
    return simpleRabbitListenerContainerFactory;    
}        
```

- 注解设置
```java
@Component
public class TestListener  {	
    //containerFactory:指定我们刚刚配置的容器
 @RabbitListener(queues = "testQueue",containerFactory = "simpleRabbitListenerContainerFactory")
    public void getMessage(Message message, Channel channel) throws Exception{
        System.out.println(new String(message.getBody(),"UTF-8"));
        System.out.println(message.getBody());
        //这里我们调用了一个下单方法  如果下单成功了 那么这条消息就可以确认被消费了
        boolean f =placeAnOrder();
        if (f){
            //传入这条消息的标识， 这个标识由rabbitmq来维护 我们只需要从message中拿出来就可以
            //第二个boolean参数指定是不是批量处理的 什么是批量处理我们待会儿会讲到
            channel.basicAck(message.getMessageProperties().getDeliveryTag(),false);
        }else {
            //当然 如果这个订单处理失败了  我们也需要告诉rabbitmq 告诉他这条消息处理失败了 可以退回 也可以遗弃 要注意的是 无论这条消息成功与否  一定要通知 就算失败了 如果不通知的话 rabbitmq端会显示这条消息一直处于未确认状态，那么这条消息就会一直堆积在rabbitmq端 除非与rabbitmq断开连接 那么他就会把这条消息重新发给别人  所以 一定要记得通知！
            //前两个参数 和上面的意义一样， 最后一个参数 就是这条消息是返回到原队列 还是这条消息作废 就是不退回了。
            channel.basicNack(message.getMessageProperties().getDeliveryTag(),false,true);
            //其实 这个API也可以去告诉rabbitmq这条消息失败了  与basicNack不同之处 就是 他不能批量处理消息结果 只能处理单条消息   其实basicNack作为basicReject的扩展开发出来的 
            //channel.basicReject(message.getMessageProperties().getDeliveryTag(),true);
        }
    }
}
```  

### 消费预取
- RMQ消费是采用轮训平均分配的，不管该消费者的消费能力是否快慢，都是平均消费，RMQ是一下子平均分配给所有消费者的。解决方案：消息预取，在使用消息预取前 要注意一定要设置为手动确认消息
- 设置
```java
@Bean
public SimpleRabbitListenerContainerFactory
simpleRabbitListenerContainerFactory(ConnectionFactory connectionFactory){
    SimpleRabbitListenerContainerFactory simpleRabbitListenerContainerFactory =
            new SimpleRabbitListenerContainerFactory();
simpleRabbitListenerContainerFactory.setConnectionFactory(connectionFactory); //手动确认消息 simpleRabbitListenerContainerFactory.setAcknowledgeMode(AcknowledgeMode.MANUAL); //设置消息预取的数量
simpleRabbitListenerContainerFactory.setPrefetchCount(1);
    return simpleRabbitListenerContainerFactory;
}
```
    
    
    
    
    
    
    
    
    
