## Springboot-rabbitmq

1. pom.xml
```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-amqp</artifactId>
</dependency>
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-web</artifactId>
</dependency>
```

2. application.yml
```
server:
  port: 8021
spring:
  #给项目来个名字
  application:
    name: springboot
  #配置rabbitMq 服务器
  rabbitmq:
    host: 192.168.52.128
    port: 5672
    username: guest
    password: guest
    #消息确认配置项
    publisher-returns: true
    # 开启消息确认机制 confirm 异步
    publisher-confirm-type: correlated
    listener:
      direct:
        # 消息开启手动确认
        acknowledge-mode: manual
        retry:
          enabled: true
          initial-interval: 5000
      simple:
        retry:
          enabled: true
          initial-interval: 5000
        acknowledge-mode: manual

  main:
    allow-bean-definition-overriding: true
```

3.  交换机配置类

3.1 Direct直连型交换机（多个接收者时，轮询消费）
```java
@Configuration
public class DirectRabbitConfig {

    //队列 起名：TestDirectQueue
    @Bean
    public Queue TestDirectQueue() {
        // durable:是否持久化,默认是false,持久化队列：会被存储在磁盘上，当消息代理重启时仍然存在，暂存队列：当前连接有效
        // exclusive:默认也是false，只能被当前创建的连接使用，而且当连接关闭后队列即被删除。此参考优先级高于durable
        // autoDelete:是否自动删除，当没有生产者或者消费者使用此队列，该队列会自动删除。
        //   return new Queue("queue",true,true,false);

        //一般设置一下队列的持久化就好,其余两个就是默认false
        return new Queue("queue",true);
    }

    //Direct交换机 起名：TestDirectExchange
    @Bean
    DirectExchange TestDirectExchange() {
        //  return new DirectExchange("TestDirectExchange",true,true);
        return new DirectExchange("exchanger",true,false);
    }

    //绑定  将队列和交换机绑定, 并设置用于匹配键：TestDirectRouting
    @Bean
    Binding bindingDirect() {
        return BindingBuilder.bind(TestDirectQueue()).to(TestDirectExchange()).with("routing");
    }

    @Bean
    DirectExchange lonelyDirectExchange() {
        return new DirectExchange("lonelyExchange");
    }
}
```

3.2 Topic交换机
```java
@Configuration
public class TopicRabbitConfig {
    //绑定键
    public final static String man = "topic.man";
    public final static String woman = "topic.woman";

    @Bean
    public Queue firstQueue() {
        return new Queue(TopicRabbitConfig.man);
    }

    @Bean
    public Queue secondQueue() {
        return new Queue(TopicRabbitConfig.woman);
    }

    @Bean
    TopicExchange exchange() {
        return new TopicExchange("topicExchange");
    }

    //将firstQueue和topicExchange绑定,而且绑定的键值为topic.man
    //这样只要是消息携带的路由键是topic.man,才会分发到该队列
    @Bean
    Binding bindingExchangeMessage() {
        return BindingBuilder.bind(firstQueue()).to(exchange()).with(man);
    }

    //将secondQueue和topicExchange绑定,而且绑定的键值为用上通配路由键规则topic.#
    // 这样只要是消息携带的路由键是以topic.开头,都会分发到该队列
    @Bean
    Binding bindingExchangeMessage2() {
        return BindingBuilder.bind(secondQueue()).to(exchange()).with("topic.#");
    }
}
```

3.3 Fanout交换机
```java
/**
 * 消息会发送给交换机上绑定的所有queue
 * 两个消费者 监听同一个queue 也是使用轮询方式接受
 **/
@Configuration
public class FanoutRabbitConfig {

    /**
     *  创建三个队列 ：fanout.A   fanout.B  fanout.C
     *  将三个队列都绑定在交换机 fanoutExchange 上
     *  因为是扇型交换机, 路由键无需配置,配置也不起作用
     */
    @Bean
    public Queue queueA() {
        return new Queue("fanout.A");
    }

    @Bean
    public Queue queueB() {
        return new Queue("fanout.B");
    }

    @Bean
    public Queue queueC() {
        return new Queue("fanout.C");
    }

    @Bean
    FanoutExchange fanoutExchange() {
        return new FanoutExchange("fanoutExchange");
    }

    @Bean
    Binding bindingExchangeA() {
        return BindingBuilder.bind(queueA()).to(fanoutExchange());
    }

    @Bean
    Binding bindingExchangeB() {
        return BindingBuilder.bind(queueB()).to(fanoutExchange());
    }

    @Bean
    Binding bindingExchangeC() {
        return BindingBuilder.bind(queueC()).to(fanoutExchange());
    }
}
```

4. 消息生产消费

4.1.1 Direct 生产者
```java
@RestController
@RequestMapping("direct")
public class DirectRabbitProductor {

    @Autowired
    RabbitTemplate rabbitTemplate;

    @RequestMapping("sendmessage")
    public String sendMessage(){
        Map<String,Object> map=new HashMap<>();
        map.put("data","hello ");
        map.put("time",new Date());
        //将消息携带绑定键值：TestDirectRouting 发送到交换机TestDirectExchange
        rabbitTemplate.convertAndSend("exchanger", "routing", map);
        return "ok";
    }
}
```
4.1.2 Direct 消费者
```java
@Component
@RabbitListener(queues = "queue")
public class DirectRabbitConsumer {

    @RabbitHandler
    public void process(Map message){
        System.out.println("RabbitConsumer 收到的消息:" + message.toString());
    }
}
```

4.2 Topic

4.2.1 生产者
```java
@RestController
@RequestMapping("topic")
public class TopicRabbitProductor {
    @Autowired
    RabbitTemplate rabbitTemplate;

    @RequestMapping("/sendmessage1")
    public String sendTopicMessage1() {
        String messageData = "message: MAN ";
        Map<String, Object> manMap = new HashMap<>();
        manMap.put("messageData", messageData);
        manMap.put("createTime", new Date());
        rabbitTemplate.convertAndSend("topicExchange", "topic.man", manMap);
        return "ok";
    }

    @RequestMapping("/sendmessage2")
    public String sendTopicMessage2() {
        String messageData = "message: woman is all ";
        Map<String, Object> womanMap = new HashMap<>();
        womanMap.put("messageData", messageData);
        womanMap.put("createTime", new Date());
        rabbitTemplate.convertAndSend("topicExchange", "topic.woman", womanMap);
        return "ok";
    }
}
```

4.2.2 消费者
```java
@Component
@RabbitListener(queues = "topic.man")
public class TopicRabbitConsumer {

    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("TopicReceiver-1 消费者收到消息  : " + testMessage.toString());
    }
}
```

4.3 fnout

4.3.1 生产者
```java
@RestController
@RequestMapping("fanout")
public class FanoutRabbitProductor {

    @Autowired
    RabbitTemplate rabbitTemplate;

    @RequestMapping("/sendFanoutMessage")
    public String sendFanoutMessage() {
        String messageData = "message: testFanoutMessage ";
        Map<String, Object> map = new HashMap<>();
        map.put("messageData", messageData);
        map.put("createTime", new Date());
        rabbitTemplate.convertAndSend("fanoutExchange", null, map);
        return "ok";
    }
}
```

4.3.2 消费者
```java
@Component
@RabbitListener(queues = "fanout.A")
public class FanoutRabbitConsumer {
    @RabbitHandler
    public void process(Map testMessage) {
        System.out.println("FanoutReceiver-A 消费者收到消息  : " +testMessage.toString());
    }
}
```


5. 消息确认

5.1 配置文件
```
raabitmq:
  #消息确认配置项
  publisher-returns: true
  # 开启消息确认机制 confirm 异步
  publisher-confirm-type: correlated
  listener:
    direct:
      # 消息开启手动确认
      acknowledge-mode: manual
      retry:
        enabled: true
        initial-interval: 5000
    simple:
      retry:
        enabled: true
        initial-interval: 5000
      acknowledge-mode: manual

``` 

5.2 消息生产者 配置发送回调
```java
@Configuration
public class RabbitCallBackConfig {
    @Bean
    public RabbitTemplate createRabbitTemplate(ConnectionFactory connectionFactory){
        RabbitTemplate rabbitTemplate = new RabbitTemplate();
        rabbitTemplate.setConnectionFactory(connectionFactory);
        //设置开启Mandatory,才能触发回调函数,无论消息推送结果怎么样都强制调用回调函数
        rabbitTemplate.setMandatory(true);
        rabbitTemplate.setConfirmCallback(new RabbitTemplate.ConfirmCallback() {
            @Override
            public void confirm(CorrelationData correlationData, boolean ack, String cause) {
                System.out.println("ConfirmCallback:     "+"相关数据："+correlationData);
                System.out.println("ConfirmCallback:     "+"确认情况："+ack);
                System.out.println("ConfirmCallback:     "+"原因："+cause);
            }
        });
        rabbitTemplate.setReturnCallback(new RabbitTemplate.ReturnCallback() {
            @Override
            public void returnedMessage(Message message, int replyCode, String replyText, String exchange, String routingKey) {
                System.out.println("ReturnCallback:     "+"消息："+message);
                System.out.println("ReturnCallback:     "+"回应码："+replyCode);
                System.out.println("ReturnCallback:     "+"回应信息："+replyText);
                System.out.println("ReturnCallback:     "+"交换机："+exchange);
                System.out.println("ReturnCallback:     "+"路由键："+routingKey);
            }
        });
        return rabbitTemplate;
    }
}
```

5.3 消息消费者 配置消息接收确认

配置消息确认的的消息监听
```java
@Configuration
public class RabbitAckConfig {
    @Autowired
    private CachingConnectionFactory connectionFactory;
    @Autowired
    private MyAckReceiver myAckReceiver;//消息接收处理类

    @Bean
    public SimpleMessageListenerContainer simpleMessageListenerContainer() {
        SimpleMessageListenerContainer container = new SimpleMessageListenerContainer(connectionFactory);
        container.setConcurrentConsumers(1);
        container.setMaxConcurrentConsumers(1);
        container.setAcknowledgeMode(AcknowledgeMode.MANUAL); // RabbitMQ默认是自动确认，这里改为手动确认消息
        //设置一个队列
        container.setQueueNames("queue");
        //如果同时设置多个如下： 前提是队列都是必须已经创建存在的
        //container.setQueueNames("queue","fanout.A");
  
        //另一种设置队列的方法,如果使用这种情况,那么要设置多个,就使用addQueues
        //container.setQueues(new Queue("queue",true));
        //container.addQueues(new Queue("TestDirectQueue2",true));
        //container.addQueues(new Queue("TestDirectQueue3",true));
        container.setMessageListener(myAckReceiver);
        return container;
    }
}
```
消息接收监听
```java
/**
 * 手动确认模式需要实现 ChannelAwareMessageListener
 * 之前的相关监听器可以先注释掉，以免造成多个同类型监听器都监听同一个队列。
 * 这里的获取消息转换，只作参考，如果报数组越界可以自己根据格式去调整。
 */
@Component
public class MyAckReceiver implements ChannelAwareMessageListener {

    @Override
    public void onMessage(Message message, Channel channel) throws Exception {
        long deliveryTag = message.getMessageProperties().getDeliveryTag();

        try {
            //因为传递消息的时候用的map传递,所以将Map从Message内取出需要做些处理
            String msg = message.toString();
            byte[] body = message.getBody();
            Object o = toObject(body);
            System.out.println(o.toString());
            String messageData=((Map<String,String>)o).get("data");
            String createTime=((Map<String, Date>)o).get("time").toString();
            System.out.println("  MyAckReceiver  data:"+messageData+"  time:"+createTime);
            System.out.println("消费的主题消息来自："+message.getMessageProperties().getConsumerQueue());
            channel.basicAck(deliveryTag, true);
//			channel.basicReject(deliveryTag, true);//为true会重新放回队列
        } catch (Exception e) {
            channel.basicReject(deliveryTag, false);
            e.printStackTrace();
        }
    }

    /**
     * 将byte数组转化为Object对象
     * @return
     */
    private Object toObject(byte[] bytes){
        Object object = null;
        try {
            ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);// 创建ByteArrayInputStream对象
            ObjectInputStream objectInputStream = new ObjectInputStream(byteArrayInputStream);// 创建ObjectInputStream对象
            object = objectInputStream.readObject();// 从objectInputStream流中读取一个对象
            byteArrayInputStream.close();// 关闭输入流
            objectInputStream.close();// 关闭输入流
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        }
        return object;// 返回对象
    }
}
```