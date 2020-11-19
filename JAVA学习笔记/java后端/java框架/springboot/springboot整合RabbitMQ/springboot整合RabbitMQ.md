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



