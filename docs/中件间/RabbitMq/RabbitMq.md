# RabbitMq

消息队列（Message Queue）是一种在应用程序之间传递消息的技术，它允许一个应用程序向另一个应用程序异步发送消息，并使得应用程序在需要的时候进行处理。消息队列通常由三个组件组成：`生产者`、`消息队列`和`消费者`。

生产者（Producer）是指向消息队列中发送消息的应用程序或系统。它们通常负责将消息发布到消息队列，并且可以在消息发送后立即返回，而无需等待消息被消费者处理完毕。

消息队列（Message Queue）是指存储消息的缓冲区，它将生产者发送的消息保存在其中，直到被消费者取走为止。消息队列通常采用FIFO（First-In-First-Out）的方式管理消息，即先进先出的原则，保证了消息的顺序性和可靠性。

消费者（Consumer）是指从消息队列中接收消息并进行处理的应用程序或系统。它们通常会长时间运行，并等待消息队列中有新的消息出现。一旦有新的消息到达，消费者会从消息队列中取走消息并进行处理。

## Work Queues

![Work Queues](https://i.328888.xyz/2023/04/02/iHtc7z.png)

工作队列（任务队列），是一种消息队列模式的实现方式，用于在分布式系统中处理耗时的任务和大量的工作。生产者发布任务到消息队列，多个消费者之间进行分配。

每个消息只能有一个消费者处理。

工作队列的主要优点是可以通过水平扩展消费者的数量来实现更高的并发处理能力，同时也能实现任务的`负载均衡`和`故障恢复`。

### 循环调度（Round-robin dispatching）

默认情况下，RabbitMq会按顺序平均将消息分配给消费者。例如，有消费者c1、c2、c3,同时有消息q1、q2、q3，RabbitMq会将q1分配给c1，q2分配给c2，q3分配给c3。

### 消息确认（Message acknowledgment）

一个消费者A如果死亡，那么它处理的消息也会丢失。消息队列如果没有等到消费者A返回的ack，会认为消息没有被处理，会进行重新排队。若此时有其它消费者B在线，则会将消息重新发送给消费者B。如果消息队列收到消费者传来的确认消息，表明消息已接收，则可以自由删除消息。

### 消息耐久性（Message durability）

若RabbitMq服务器停止，则消息还是会丢失。

```java
channel.queueDeclare("hello", true, false, false, null);
```

其中第二个参数可以设置持久性。

### 公平分配（Fair dispatch）

防止出现一个消费者一直处理任务重的消息，另一个消费者只处理任务轻的消息。

```java
channel.basicQos(1);
```

设置消费者最多接收多少条消息。消费者在接收到队列里的消息但没有返回确认结果之前,队列不会将新的消息分发给该消费者。

如下图，消息队列中有消息1，2，3，4，按照循环调度，WorkerA获得1，3消息，WorkerB获得2，4消息，但WorkerB处理2的时间较长，且设置了消费者最多同时接收一条消息，所以编号为4的消息分配给了WorkerA。

`WorkerA`

![WorkerA](https://i.328888.xyz/2023/04/02/iHtDsH.png)

`WorkerB`

![WorkerB](https://i.328888.xyz/2023/04/02/iHtAcF.png)

## Publish/Subscribe

向多个消费者发送消息，发布订阅模式。

![Publish/Subscribe](https://i.328888.xyz/2023/04/02/iHWihV.png)

相比于工作队列多了一个交换机(exchange)，多个队列绑定到交换机上。生产者将消息发送到交换机，再由交换机路由到绑定的一个或多个队列中。

`临时队列`一种自动删除的队列，它是一个短暂存在的队列，当消费者断开连接时自动删除。

```java
String queueName = channel.queueDeclare().getQueue();
```

![](https://i.328888.xyz/2023/04/02/iHt9IQ.png)

![](https://i.328888.xyz/2023/04/02/iHt1WJ.png)

## 路由模式（Routing）



![Routing](https://i.328888.xyz/2023/04/02/iHRVnH.png)

RabbitMq有三种常见的路由模式：

- Fanout：消息通过交换机发送给所有绑定的队列，所有绑定的队列都能接收到消息。
- Direct：当消息的路由键与队列绑定的绑定键相同时，消息才会发送到该队列。
- Topic：与Direct类似，但Topic可以通过通配符进行路由键匹配。

```java
channel.queueBind(QUEUE_NAME, EXCHANGE_NAME, ROUTING_KEY);
```

在Java中，绑定队列时可以通过`ROUTING_KEY`参数进行路由键绑定。

```java
channel.basicPublish(EXCHANGE_NAME, ROUTING_KEY, null, message.getBytes());
```

发送消息时，同样可以通过`ROUTING_KEY`赋予消息路由键。

## Topic

Topic模式是一种高级的路由模式，它可以根据消息的路由键和通配符将消息路由到多个队列中。

![Topic](https://i.328888.xyz/2023/04/02/iHRPWA.png)

`*`可以代替一个单词

`#`可以代替零个或多个单词

- `quick.orange.pig`:将匹配`*.orange.*`,发送到Q1队列；
- `quick.orange.rabbit`：将匹配`*.orange.*`和`*.*.rabbit`,发送到Q1和Q2队列；
- `orange`或`quick.orange.new.rabbit`：没有匹配的绑定键；
- `lazy.orange.new.rabbit`：将匹配`layz.#`，`#`可以代替零个或多个单词，所以即时lazy后接多个单词也是匹配的。而`*.orange.*`是用`*`，只能代替一个单词，所以不匹配。

## 远程过程调用（Remote procedure call (RPC)）

在RPC模式中，客户端发送请求消息时，需要指定一个唯一的回调队列名称，并在消息中包含这个队列名称，服务器在处理完请求后，将响应消息发送回这个回调队列。客户端在发送请求消息后，会一直等待接收回调队列中的响应消息。

![](https://i.328888.xyz/2023/04/02/iHpwcF.png)

##  AMQP

AMQP（高级消息队列协议）是一个网络协议。它支持符合要求的客户端应用（application）和消息中间件代理（messaging middleware broker）之间进行通信。

