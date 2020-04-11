---
title: Message Queue ——RabbitMQ
date: 2019-11-11 10:20:33
categories:
    - Basic Component
tags:
    - RabbitMQ
    - MessageQueue
description: "消息队列在开发中十分重要，本文以java为开发语言，介绍并实战了消息中间件rabbitmq，涉及了三种模式：fanout、direct、topic"
fancybox: true  # 图片浏览器
toc: true       # 文章目录
original: true  # 文末版权信息 
comments: true  # 文末评论
share: true     # 分享
---

## 消息中间件
消息中间件有很多种，如 activemq, rabbitmq 等等。

## RabbitMQ 模式概述
>  rabbitmq 使用的是一种叫做 AMQP 的协议来通信。 AMQP 是 advanced Message Queuing Protocol 的缩写。简单地说，通过这种协议，可以处理更为复杂的业务需求。

基于 AMQP 这种协议，可以实现如下各种模式。

### 消息路由过程
与 ActiveMQ 拿到消息就直接放在队列等待消费者拿走不同， Rabbit 拿到消息之后，会先交给 交换机 （Exchange）, 然后交换机再根据预先设定的不同绑定( Bindings )策略，来确定要发给哪个队列。
如图所示，比起 ActiveMQ 多了 Exchange 和 Bindings。
正是由于有了    Exchange 和 Bindings， RabbitMQ 就可以灵活地支撑各种模式。
![](RabbitMQ_3.png)
RabbitMQ提供了四种Exchange模式：fanout,direct,topic,header 。 header模式在实际使用中较少，一般不作考虑。

#### Fanout 模式
fanout 模式就是广播模式。消息来了，会发给所有的队列。 
![](RabbitMQ_4.png)

#### Direct 模式
Direct 模式就是指定队列模式，消息来了，只发给指定的 Queue, 其他Queue 都收不到。 
![](RabbitMQ_5.png)

#### Topic 模式
主题模式，注意这里的主题模式，和 ActivityMQ 里的不一样。 ActivityMQ 里的主题，更像是广播模式。 
那么这里的主题模式是什么意思呢？ 如图所示消息来源有： 美国新闻，美国天气，欧洲新闻，欧洲天气。 
如果你想看 <font color="#FFFF66">美国</font>主题： 那么就会收到 <font color="#FFFF66">美国</font>新闻，<font color="#FFFF66">美国</font>天气。 
如果你想看 <font color="#FFFF66">新闻</font>主题： 那么就会收到 <font color="#FFFF66">美国</font>新闻，<font color="#FFFF66">欧洲</font>新闻。 
如果你想看 <font color="#FFFF66">天气</font>主题： 那么就会收到 <font color="#FFFF66">美国</font>天气，<font color="#FFFF66">欧洲</font>天气。 
如果你想看 <font color="#FFFF66">欧洲</font>主题： 那么就会收到 <font color="#FFFF66">欧洲</font>新闻，<font color="#FFFF66">欧洲</font>天气。 
![](RabbitMQ_6.png)

        
### Fanout模式
#### pom 中加入依赖
```xml
    <dependency>
            <groupId>com.rabbitmq</groupId>
            <artifactId>amqp-client</artifactId>
            <version>3.6.5</version>
     </dependency>
    <dependency>
        <groupId>cn.hutool</groupId>
        <artifactId>hutool-all</artifactId>
        <version>4.3.1</version>
    </dependency>
```

#### 判断服务是否启动类
```java
package com.how2j.testrabbitmq;

import javax.swing.JOptionPane;
import cn.hutool.core.util.NetUtil;

public class RabbitMQUtil {

    public static void main(String[] args) {
        checkServer();
    }
    public static void checkServer() {
        if(NetUtil.isUsableLocalPort(15672)) {
            JOptionPane.showMessageDialog(null, "RabbitMQ 服务器未启动 ");
            System.exit(1);
        }
        else {
			 System.out.println("服务器已启动");
		}
    }
}
```

#### 接收消息Customer
```java
package com.how2j.testrabbitmq;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

import cn.hutool.core.util.RandomUtil;

public class FanoutCustomer {
    public final static String EXCHANGE_NAME="fanout_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        //为当前消费者取随机名
        String name = "consumer-"+ RandomUtil.randomString(5);

        //判断服务器是否启动
        RabbitMQUtil.checkServer();
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ地址
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();
        //交换机声明（参数为：交换机名称；交换机类型）
        channel.exchangeDeclare(EXCHANGE_NAME,"fanout");
        //获取一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //队列与交换机绑定（参数为：队列名称；交换机名称；routingKey忽略）
        channel.queueBind(queueName,EXCHANGE_NAME,"");

        System.out.println(name +" 等待接受消息");
        //DefaultConsumer类实现了Consumer接口，通过传入一个频道，
        // 告诉服务器我们需要那个频道的消息，如果频道中有消息，就会执行回调函数handleDelivery
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(name + " 接收到消息 '" + message + "'");
            }
        };
        //自动回复队列应答 -- RabbitMQ中的消息确认机制
        channel.basicConsume(queueName, true, consumer);
    }
}
```

#### 发送消息Producer
```java
package com.how2j.testrabbitmq;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消息生成者
 */
public class FanoutProducer {
    public final static String EXCHANGE_NAME="fanout_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        RabbitMQUtil.checkServer();

        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ相关信息
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "fanout");

        for (int i = 0; i < 100; i++) {
            String message = "direct 消息 " +i;
            //发送消息到队列中
            channel.basicPublish(EXCHANGE_NAME, "", null, message.getBytes("UTF-8"));
            System.out.println("发送消息： " + message);

        }
        //关闭通道和连接
        channel.close();
        connection.close();
    }
}
```

#### 先运行两次 FanoutCustomer 启动两个消费者，再运行一次 FanoutProducer 启动生产者。效果如下：
![](RabbitMQ_7.png)
        
### Direct模式
#### 除了生产者和消费者不同，其余与Fanout一致。
#### 接收消息Customer
```java
package com.how2j.testrabbitmq;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

import cn.hutool.core.util.RandomUtil;

public class DirectCustomer {
    private final static String QUEUE_NAME = "direct_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        //为当前消费者取随机名
        String name = "consumer-"+ RandomUtil.randomString(5);

        //判断服务器是否启动
        RabbitMQUtil.checkServer();
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ地址
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();
        //声明要关注的队列
        channel.queueDeclare(QUEUE_NAME, false, false, true, null);
        System.out.println(name +" 等待接受消息");
        //DefaultConsumer类实现了Consumer接口，通过传入一个频道，
        // 告诉服务器我们需要那个频道的消息，如果频道中有消息，就会执行回调函数handleDelivery
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(name + " 接收到消息 '" + message + "'");
            }
        };
        //自动回复队列应答 -- RabbitMQ中的消息确认机制
        channel.basicConsume(QUEUE_NAME, true, consumer);
    }
}
```
#### 发送消息Producer
```java
package com.how2j.testrabbitmq;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消息生成者
 */
public class DirectProducer {
    public final static String QUEUE_NAME="direct_queue";

    public static void main(String[] args) throws IOException, TimeoutException {
        RabbitMQUtil.checkServer();

        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ相关信息
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();

        for (int i = 0; i < 100; i++) {
            String message = "direct 消息 " +i;
            //发送消息到队列中
            channel.basicPublish("", QUEUE_NAME, null, message.getBytes("UTF-8"));
            System.out.println("发送消息： " + message);

        }
        //关闭通道和连接
        channel.close();
        connection.close();
    }
}
```
#### 效果
![](RabbitMQ_8.png)
        
### Topic模式
#### 除了生产者和消费者不同，其余与Fanout一致。
#### 接收消息Customer
##### USA
```java
package com.how2j.testrabbitmq;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class TopicCustomerUSA {
    public final static String EXCHANGE_NAME="topics_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        //为当前消费者取名称
        String name = "consumer-usa";

        //判断服务器是否启动
        RabbitMQUtil.checkServer();
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ地址
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();
        //交换机声明（参数为：交换机名称；交换机类型）
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        //获取一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //接受 USA 信息

        channel.queueBind(queueName, EXCHANGE_NAME, "usa.*");
        System.out.println(name +" 等待接受消息");
        //DefaultConsumer类实现了Consumer接口，通过传入一个频道，
        // 告诉服务器我们需要那个频道的消息，如果频道中有消息，就会执行回调函数handleDelivery
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(name + " 接收到消息 '" + message + "'");
            }
        };
        //自动回复队列应答 -- RabbitMQ中的消息确认机制
        channel.basicConsume(queueName, true, consumer);
    }
}
```
##### News
```java
package com.how2j.testrabbitmq;
import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.AMQP;
import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;
import com.rabbitmq.client.Consumer;
import com.rabbitmq.client.DefaultConsumer;
import com.rabbitmq.client.Envelope;

public class TopicCustomerNews {
    public final static String EXCHANGE_NAME="topics_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        //为当前消费者取名称
        String name = "consumer-news";

        //判断服务器是否启动
        RabbitMQUtil.checkServer();
        // 创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ地址
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();
        //交换机声明（参数为：交换机名称；交换机类型）
        channel.exchangeDeclare(EXCHANGE_NAME,"topic");
        //获取一个临时队列
        String queueName = channel.queueDeclare().getQueue();
        //接受 USA 信息

        channel.queueBind(queueName, EXCHANGE_NAME, "*.news");
        System.out.println(name +" 等待接受消息");
        //DefaultConsumer类实现了Consumer接口，通过传入一个频道，
        // 告诉服务器我们需要那个频道的消息，如果频道中有消息，就会执行回调函数handleDelivery
        Consumer consumer = new DefaultConsumer(channel) {
            @Override
            public void handleDelivery(String consumerTag, Envelope envelope,
                                       AMQP.BasicProperties properties, byte[] body)
                    throws IOException {
                String message = new String(body, "UTF-8");
                System.out.println(name + " 接收到消息 '" + message + "'");
            }
        };
        //自动回复队列应答 -- RabbitMQ中的消息确认机制
        channel.basicConsume(queueName, true, consumer);
    }
}
```
#### 发送消息Producer
```java
package com.how2j.testrabbitmq;

import java.io.IOException;
import java.util.concurrent.TimeoutException;

import com.rabbitmq.client.Channel;
import com.rabbitmq.client.Connection;
import com.rabbitmq.client.ConnectionFactory;

/**
 * 消息生成者
 */
public class TopicProducer {
    public final static String EXCHANGE_NAME="topics_exchange";

    public static void main(String[] args) throws IOException, TimeoutException {
        RabbitMQUtil.checkServer();

        //创建连接工厂
        ConnectionFactory factory = new ConnectionFactory();
        //设置RabbitMQ相关信息
        factory.setHost("localhost");
        //创建一个新的连接
        Connection connection = factory.newConnection();
        //创建一个通道
        Channel channel = connection.createChannel();

        channel.exchangeDeclare(EXCHANGE_NAME, "topic");

        String[] routing_keys = new String[] { "usa.news", "usa.weather",
                "europe.news", "europe.weather" };
        String[] messages = new String[] { "美国新闻", "美国天气",
                "欧洲新闻", "欧洲天气" };

        for (int i = 0; i < routing_keys.length; i++) {
            String routingKey = routing_keys[i];
            String message = messages[i];
            channel.basicPublish(EXCHANGE_NAME, routingKey, null, message
                    .getBytes());
            System.out.printf("发送消息到路由：%s, 内容是: %s%n ", routingKey,message);

        }

        //关闭通道和连接
        channel.close();
        connection.close();
    }
}
```
#### 效果
![](RabbitMQ_9.png)

Demo代码可以 Email 联系我。