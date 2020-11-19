# ==RabbitMQ==

### 1.直接连接queue模式

(不是springboot项目)

==消息生产者==

1.添加pom依赖

```xml
		<dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.9.0</version>
        </dependency>
```

2.编写代码

```java
public class Provider {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        //设置连接rabbitmq主机
        connectionFactory.setHost("101.201.150.92");
        //设置rabbitmq端口
        connectionFactory.setPort(5672);
        //设置虚拟主机
        connectionFactory.setVirtualHost("/mq");
        //设置用户名
        connectionFactory.setUsername("java");
        //设置用户密码
        connectionFactory.setPassword("java");
        //获得rabbitmq连接
        Connection connection = connectionFactory.newConnection();
        //获取连接通道
        Channel channel = connection.createChannel();
        //消息通道绑定消息队列
        /**
         * 参数一 queue ：参数名
         * 参数二 durable : 是否持久化
         * 参数三 exclusive ： 是否独占一个队列
         * 参数四 autoDelete ： 队列中消息读取完是否自动删除
         * 参数五 arguments ： 额外参数
         */
        channel.queueDeclare("hello",false,false,false,null);
        //发送消息
        /**
         * 参数一 exchange ： 交换机
         * 参数二 routing ： 路由键   如果没有交换机则路由键就是队列名
         * 参数三 props ： 消息头
         * 参数四 body ： 消息体
         */
        channel.basicPublish("","hello",null,"你好啊".getBytes());
        //关闭通道
        channel.close();
        //关闭连接
        connection.close();
    }
}
```



==消息消费者==

1.添加pom依赖

```xml
		<dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>5.9.0</version>
        </dependency>
```

2.编写代码

```java
public class Consumer {
    public static void main(String[] args) throws IOException, TimeoutException {
        ConnectionFactory connectionFactory = new ConnectionFactory();
        connectionFactory.setHost("101.201.150.92");
        connectionFactory.setPort(5672);
        connectionFactory.setVirtualHost("/mq");
        connectionFactory.setUsername("java");
        connectionFactory.setPassword("java");
        Connection connection = connectionFactory.newConnection();
        Channel channel = connection.createChannel();
        /**
         * 参数一 queue ： 消息队列名
         * 参数二 autoAck ： 收到消息自动回复
         * 参数三 callback ： 回调函数
         */
        //接收信息
        channel.basicConsume("hello",true,new DefaultConsumer(channel){
            @Override
            /**
             * 参数一 consumerTag ： 消费标签
             * 参数二 envelope ： 消息信封
             * 参数三 properties ： 消息头信息
             * 参数四 body ： 消息体
             */
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                String str = new String(body);
                System.out.println(str);
            }
        });
    }
}
```

==rabbitmq工具类提供了直接获取连接对象，获取通道对象，关闭通道和连接的方法==

```java
public class MQUtil {
    private static ConnectionFactory connectionFactory = new ConnectionFactory();
    //获取连接
    public static Connection getConnection(String host,int port,String virtualHost,String username,String password) {
        try {
            connectionFactory.setHost(host);
            //设置rabbitmq端口
            connectionFactory.setPort(port);
            //设置虚拟主机
            connectionFactory.setVirtualHost(virtualHost);
            //设置用户名
            connectionFactory.setUsername(username);
            //设置用户密码
            connectionFactory.setPassword(password);
            //获得rabbitmq连接
            Connection connection = connectionFactory.newConnection();
            return connection;
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
    //获得通道
    public static Channel getChannel(Connection connection) {
        try {
            return connection.createChannel();
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
    //获得通道
    public static Channel getChannel(String host,int port,String virtualHost,String username,String password){
        try {
            return getConnection(host,port,virtualHost,username,password).createChannel();
        }catch (Exception e){
            e.printStackTrace();
            return null;
        }
    }
    //关闭通道和连接
    public static void closeChannelAndConnection(Channel channel,Connection connection){
        try {
            if (channel != null){
                channel.close();
            }
            if (connection != null){
                connection.close();
            }
        }catch (Exception e){
            e.printStackTrace();
        }
    }
}
```

### 2.WORK  QUEUE 

​		work queue模式和单点差不多，唯一的差别就是work queue模式两个以上消费者。多个消费者同时监听一个队列，如果使用自动消息确认的方式，那么每个消息会按轮巡算法分配给每一个消息消费者，并且每一个消息消费者拿到消息立即确认消息，此时消息实际上还没处理完，这种work queue模式是把消息平均分配给每个消息消费者，这种方式有个最大的缺点就是，如果有一个消息消费者处理消息很慢，那么也只能慢慢处理，别的消息消费者不会帮他处理，并且拿到消息就确认还有个缺点，就是拿到的消息还没处理完，但是消息消费者宕机了就会导致消息丢失。所以我们一般采用手动消息确认的方式，这样实现了能者多劳，避免消息丢失的问题。

==手动消息确认方式代码(这些说的都是消息消费者，和消息生产者无关)==

```java
public class consumer1 {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.basicQos(1);//每次消费一个消息
        //设置不自动确认消息
        channel.basicConsume("work",false,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
                //手动确认消息
                //第一个参数为消息标签 第二个参数是 是否同时确认多个消息
                channel.basicAck(envelope.getDeliveryTag(),false);
            }
        });
    }
}
```

### 3.广播模式

发送消息给所有绑定此交换机的队列

==生产者代码==

```java
public class Provider {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        //声明交换机
        /**
       	 *第一个参数为交换机名字
       	 *第二个参数为交换机类型
       	 */
        channel.exchangeDeclare("fanout","fanout");
        //发送消息给交换机
        channel.basicPublish("fanout","",null,"你好啊".getBytes());
        MQUtil.closeChannelAndConnection(channel,connection);
    }
}
```



==消费者代码==

```java
public class Consumer1 {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        //获得一个随机名字的临时队列.
        String queueName = channel.queueDeclare().getQueue();
        //消费者绑定队列和交换机,消费者也可以声明队列。只要满足下面视图即可
        //生产者-->通道-->交换机-->队列-->通道-->消费者,只要满足这条线可以随意分配消费者和生产者代码
        channel.queueBind(queueName,"fanout","");
        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println("我是消费者1"+new String(body));
            }
        });
    }
}
```

### 4.路由模式

发送消息给带有指定路由键的队列

==生产者代码==

```java
public class Provider {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("direct","direct");
        channel.basicPublish("direct","error",null,"error".getBytes());
        MQUtil.closeChannelAndConnection(channel,connection);
    }
}
```

==消费者代码==

```java
public class Consumer1 {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("direct","direct");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName,"direct","info");
        channel.queueBind(queueName,"direct","error");
        channel.queueBind(queueName,"direct","warning");
        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });
    }
}
```

### 5.Topic模式

==生产者代码==

```java
public class Provider {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("topic","topic");
        channel.basicPublish("topic","user.save",null,"user".getBytes());
        MQUtil.closeChannelAndConnection(channel,connection);
    }
}
```

==消费者代码==

```java
public class Consumer1 {
    public static void main(String[] args) throws IOException {
        Connection connection = MQUtil.getConnection();
        Channel channel = connection.createChannel();
        channel.exchangeDeclare("topic","topic");
        String queueName = channel.queueDeclare().getQueue();
        channel.queueBind(queueName,"topic","*.user.#");
        channel.basicConsume(queueName,true,new DefaultConsumer(channel){
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope, AMQP.BasicProperties properties, byte[] body) throws IOException {
                System.out.println(new String(body));
            }
        });
    }
}
```

### springboot整合RabbitMQ

==helloworld模式==

==生产者==

```java
@Test
    void contextLoads() {
        rabbitTemplate.convertAndSend("hello","hello world");
    }
```

==消费者==

```java
@RabbitListener(queuesToDeclare = @Queue("hello"))
    public void consumer(String message){
        System.out.println(message);
    }
```

==work queue模式==

和helloworld模式一样只是多几个消费者

==fanout模式==

==生产者==

```java
@Test
    public void test1(){
        rabbitTemplate.convertAndSend("fanout","","fanout类型交换机");
    }
```

==消费者==

```java
@RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "fanout",type = "fanout")
    ))
    public void consumer1(String message){
        System.out.println(message);
    }
```

==路由模式==

==生产者==

```java
@Test
    public void test2(){
        rabbitTemplate.convertAndSend("direct","info","我是路由交换机我发送的是info路由");
    }
```

==消费者==

```java
@Component(value = "consumerDirect")
public class Consumer {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(value = "direct",type = "direct"),
        //绑定路由键，可以绑定多个
            key = {"info","error","warning"}
    ))
    public void consumer(String message){
        System.out.println(message);
    }
}
```

==动态路由==

==生产者==

```java
@Test
    public void test3(){
        rabbitTemplate.convertAndSend("topic","user.save","hahaha.user.save");
    }
```

==消费者==

```java
@Component(value = "componentTopic")
public class Consumer {
    @RabbitListener(bindings = @QueueBinding(
            value = @Queue,
            exchange = @Exchange(name = "topic",type = "topic"),
        //多个匹配规则满足一个就可以收到消息
            key = {"user.#","*.user.#"}

    ))
    public void consumer(String message){
        System.out.println(message);
    }
}
```



