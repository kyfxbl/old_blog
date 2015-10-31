title: jms基础
date: 2013-09-24 11:07
categories: java 
---
Java Message Service是java ee的规范之一，可以用来发送异步消息，在某些场景下，可以作为不同系统，或者不同模块之间的集成方式。可以类比为通过数据库来集成的方式，模块A完成逻辑以后，往数据库插入一条记录，模块B定时轮询数据库，如果查到相应的记录，就进行处理。jms集成实际上思路是差不多的，只是功能更强，并且提供了标准的API支持，而且也可以避免反复轮询数据库或者读取文件的I/O操作，对系统的整体性能会有提升。本文介绍JMS的一些基础知识
<!--more-->

JMS的主要优点，首先是可以使2个系统或模块实现松耦合，模块A不需要直接调用模块B，只需要往jms provider上发送一条约定格式的消息，模块B收到这条消息，进行后续的业务处理 

其次，jms方式是异步的，意味着模块A发送消息之后，不需要等待模块B或者jms provider的响应，自身的业务逻辑可以继续 

jms技术对应的规范是jsr914，规范的实现称为jms provider，常见的实现有ActiveMQ、JBoss MQ、IBM Websphere MQ等。本文以ActiveMQ举例 

# ActiveMQ使用 

ActiveMQ（其他的jms provider也差不多）安装之后，目录结构是这样的： 

![](http://dl.iteye.com/upload/attachment/0072/1309/c312bba4-1628-3e22-95d3-3da1843b391c.png)

运行bin目录下的activemq.bat，会根据默认配置，启动一个broker。各种jms实现，好像都有broker的概念 

启动之后，会占用至少2个端口，默认的是61616和8161 

61616是等待jms client的连接，8161是ActiveMQ自带的一个web应用 
```
http://localhost:8161/demo
```

可以看到各种官方提供的例子 

```
http://localhost:8161/admin
```

是ActiveMQ的管理控制台 
![](http://dl.iteye.com/upload/attachment/0072/1312/84e11d71-540f-3e7d-81d6-c1dd048453b2.png)

这里可以对队列进行各种操作，比如发送消息，查看消息，清空队列等等 

ActiveMQ即使在不编程的情况下，也可以通过这种方式来使用。当然，大部分情况，还是需要针对jms client进行编程的 

# jms基本概念 

前面说过，jms的实现，称为jms provider，可以认为是jms的服务器 

jms的客户端，需要开发人员自行开发，称为jms client 

jms的消息机制有2种模型，一种是Point to Point，表现为队列的形式。发送的消息，只能被一个接收者取走 

另一种是Topic，可以被多个订阅者订阅，类似于群发 

![](http://dl.iteye.com/upload/attachment/0072/1319/5520cd1a-323b-3ea5-badb-86b92fdc5d4f.png)

ConnectionFactory，用于jms client获取与jms provider的连接。不同的jms产品，对这个接口有不同的实现，比如说ActiveMQ，这个接口的实现类是ActiveMQConnectionFactory 

Connection，是由ConnectionFactory产生的，表示jms client与jms provider的连接 

Session，是由Connection产生的，表示一个会话。Session是关键组件，Message、Producer/Consumer、Destination都是在Session上创建的 

Message，这个组件很好理解，就是传输的消息，里面包括head、properties、body，其中head是必选的 

Destination，是消息源，对发送者来说，就是消息发到哪里；对接收者来说，就是从哪里取消息。Destination有2个子接口，Queue和Topic，分别对应上面提到的2种模型 

Message Producer，是消息发送者，创建这个组件的代码类似：
```
Destination dest = session.createQueue("dotaQueue");// 消息目的地
MessageProducer producer = session.createProducer(dest);// 消息发送者
```

可以注意到，这里需要把Destination作为参数，传入createProducer()方法，这说明消息发送者是绑定到Destination上的，这个发送者发送的消息，会发送到这个绑定的Destination上 

Message Consumer，是消息接收者，和Message Producer是相反的一种组件 

# 代码实例 

这里是基于ActiveMQ进行开发，所以需要导入ActiveMQ提供的jar包。不过开发时，应该尽量针对jms接口进行开发，不依赖特定的实现 

例子是用main函数跑的，没有在java ee容器里跑，所以没有办法依赖JNDI拿到ConnectionFactory的实例，只能手工创建ActiveMQConnectionFactory，所以和ActiveMQ的实现耦合了，没有办法连到别的jms实现上。如果实际的代码，用JNDI或者spring来获取ConnectionFactory的实例的话，那就可以仅针对接口编程，连接到任意jms provider了 

为了简单起见，例子也没有用到spring，实际上spring对jms client提供了很好的支持

开发环境只要导入activemq-all-5.6.0.jar就可以了 

![](http://dl.iteye.com/upload/attachment/0072/1326/78e8066f-8efb-3af9-bf89-fca8ae8e16be.png)

里面已经包括了jms API、activemq-core、javaee-management API等必须的class 

首先是Message Producer的例子：

```
public class Main {

	public static void main(String[] args) throws JMSException {

		String jmsProviderAddress = "tcp://localhost:61616";// 地址

		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
				jmsProviderAddress);// 连接器

		Connection connection = connectionFactory.createConnection();// 创建连接

		Session session = connection.createSession(false,
				Session.AUTO_ACKNOWLEDGE);// 打开会话

		Destination dest = session.createQueue("demoQueue");// 消息目的地

		MessageProducer producer = session.createProducer(dest);// 消息发送者

		Message message = session.createTextMessage("hello world");// 消息

		producer.send(message);// 发送

		producer.close();// 关闭
		session.close();
		connection.close();
	}
}
```

代码很简单，可以参考上面的图，各组件的关系是比较清楚的 

然后是Message Consumer的例子：

```
public static void main(String[] args) throws JMSException {

		String jmsProviderAddress = "tcp://localhost:61616";// 地址

		ConnectionFactory connectionFactory = new ActiveMQConnectionFactory(
				jmsProviderAddress);// 连接器

		Connection connection = connectionFactory.createConnection();// 创建连接

		Session session = connection.createSession(false,
				Session.AUTO_ACKNOWLEDGE);// 打开会话

		String destinationName = "demoQueue";

		Destination dest = session.createQueue(destinationName);// 消息目的地

		MessageConsumer consumer = session.createConsumer(dest);

		connection.start();

		Message message = consumer.receive();

		TextMessage textMessage = (TextMessage) message;

		String text = textMessage.getText();

		System.out.println("从ActiveMQ取回一条消息: " + text);

		consumer.close();
		session.close();
		connection.close();
	}
```
和MessageProducer的代码基本类似，实际中一般会实现javax.jms.MessageListener接口，这样就不需要手工调用receive()方法