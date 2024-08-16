# 六种工作模式

## simple简单模式

消费者拿走一个mq删除一个，无法保证消费者正确处理数据

一般用来聊天

## work工作模式

多个消费者共同争抢争抢当前的消息队列内容，谁先拿到谁负责消费消息

用来红包

## publish/subscribe发布订阅

生产者将消息发送给交换机，由交换机发送信息给每个消息队列，对应消息队列的消费者拿到消息进行消费

## routing路由模式

在发布订阅模式上增加了路由规则，只将消息发送给匹配规则的消息队列

## topic 主题模式

交换机根据key的模糊匹配规则，匹配到对应的队列

# 操作mq

## 初始化库

~~~go

import (
	"log"
	"github.com/streadway/amqp"
) 

// MQURL 格式 amqp://账号：密码@rabbitmq服务器地址：端口号/vhost (默认是5672端口)
// 端口可在 /etc/rabbitmq/rabbitmq-env.conf 配置文件设置，也可以启动后通过netstat -tlnp查看
const MQURL = "amqp://admin:huan91uncc@172.21.138.131:5672/"

type RabbitMQ struct {
	Conn    *amqp.Connection
	Channel *amqp.Channel
	// 队列名称
	QueueName string
	// 交换机
	Exchange string
	// routing Key
	RoutingKey string
	//MQ链接字符串
	Mqurl string
}

// 创建结构体实例
func NewRabbitMQ(queueName, exchange, routingKey string) *RabbitMQ {
	rabbitMQ := RabbitMQ{
		QueueName:  queueName,
		Exchange:   exchange,
		RoutingKey: routingKey,
		Mqurl:      MQURL,
	}
	var err error
	//创建rabbitmq连接
	rabbitMQ.Conn, err = amqp.Dial(rabbitMQ.Mqurl)
	checkErr(err, "创建连接失败")

	//创建Channel
	rabbitMQ.Channel, err = rabbitMQ.Conn.Channel()
	checkErr(err, "创建channel失败")

	return &rabbitMQ

}

// 释放资源,建议NewRabbitMQ获取实例后 配合defer使用
func (mq *RabbitMQ) ReleaseRes() {
	mq.Conn.Close()
	mq.Channel.Close()
}

func checkErr(err error, meg string) {
	if err != nil {
		log.Fatalf("%s:%s\n", meg, err)
	}
}
~~~

## 生产者

~~~go
package main

import (
	"fmt"
	"mq/rabbitMq"

	"github.com/streadway/amqp"
)

//生产者发布流程
func main() {
	// 初始化mq
	mq := rabbitMq.NewRabbitMQ("queue_publisher", "exchange_publisher", "key1")
	defer mq.ReleaseRes() // 完成任务释放资源

	// 1.声明队列
	/*
		如果只有一方声明队列，可能会导致下面的情况：
			a)消费者是无法订阅或者获取不存在的MessageQueue中信息
			b)消息被Exchange接受以后，如果没有匹配的Queue，则会被丢弃

		为了避免上面的问题，所以最好选择两方一起声明
		ps:如果客户端尝试建立一个已经存在的消息队列，Rabbit MQ不会做任何事情，并返回客户端建立成功的
	*/
	_, err := mq.Channel.QueueDeclare( // 返回的队列对象内部记录了队列的一些信息，这里没什么用
		mq.QueueName, // 队列名
		true,         // 是否持久化
		false,        // 是否自动删除(前提是至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。注意：生产者客户端创建这个队列，或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列)
		false,        // 是否为排他队列（排他的队列仅对“首次”声明的conn可见[一个conn中的其他channel也能访问该队列]，conn结束后队列删除）
		false,        // 是否阻塞
		nil,          //额外属性（我还不会用）
	)
	if err != nil {
		fmt.Println("声明队列失败", err)
		return
	}

	// 2.声明交换器
	err = mq.Channel.ExchangeDeclare(
		mq.Exchange, //交换器名
		"topic",     //exchange type：一般用fanout、direct、topic
		true,        // 是否持久化
		false,       //是否自动删除（自动删除的前提是至少有一个队列或者交换器与这和交换器绑定，之后所有与这个交换器绑定的队列或者交换器都与此解绑）
		false,       //设置是否内置的。true表示是内置的交换器，客户端程序无法直接发送消息到这个交换器中，只能通过交换器路由到交换器这种方式
		false,       // 是否阻塞
		nil,         // 额外属性
	)
	if err != nil {
		fmt.Println("声明交换器失败", err)
		return
	}

	// 3.建立Binding(可随心所欲建立多个绑定关系)
	err = mq.Channel.QueueBind(
		mq.QueueName,  // 绑定的队列名称
		mq.RoutingKey, // bindkey 用于消息路由分发的key
		mq.Exchange,   // 绑定的exchange名
		false,         // 是否阻塞
		nil,           // 额外属性
	)
	// err = mq.Channel.QueueBind(
	// 	mq.QueueName,  // 绑定的队列名称
	// 	"routingkey2", // bindkey 用于消息路由分发的key
	// 	mq.Exchange,   // 绑定的exchange名
	// 	false,         // 是否阻塞
	// 	nil,           // 额外属性
	// )
	if err != nil {
		fmt.Println("绑定队列和交换器失败", err)
		return
	}

	// 4.发送消息
	mq.Channel.Publish(
		mq.Exchange,   // 交换器名
		mq.RoutingKey, // routing key
		false,         // 是否返回消息(匹配队列)，如果为true, 会根据binding规则匹配queue，如未匹配queue，则把发送的消息返回给发送者
		false,         // 是否返回消息(匹配消费者)，如果为true, 消息发送到queue后发现没有绑定消费者，则把发送的消息返回给发送者
		amqp.Publishing{ // 发送的消息，固定有消息体和一些额外的消息头，包中提供了封装对象
			ContentType: "text/plain",           // 消息内容的类型
			Body:        []byte("hello jochen"), // 消息内容
		},
	)
}
~~~

## 消费者

~~~go
package main

import (
	"fmt"
	"mq/rabbitMq"
)

// 消费者订阅
func main() {
	// 初始化mq
	mq := rabbitMq.NewRabbitMQ("queue_publisher", "exchange_publisher", "key1")
	defer mq.ReleaseRes() // 完成任务释放资源

	// 1.声明队列（两端都要声明，原因在生产者处已经说明）
	_, err := mq.Channel.QueueDeclare( // 返回的队列对象内部记录了队列的一些信息，这里没什么用
		mq.QueueName, // 队列名
		true,         // 是否持久化
		false,        // 是否自动删除(前提是至少有一个消费者连接到这个队列，之后所有与这个队列连接的消费者都断开时，才会自动删除。注意：生产者客户端创建这个队列，或者没有消费者客户端与这个队列连接时，都不会自动删除这个队列)
		false,        // 是否为排他队列（排他的队列仅对“首次”声明的conn可见[一个conn中的其他channel也能访问该队列]，conn结束后队列删除）
		false,        // 是否阻塞
		nil,          // 额外属性（我还不会用）
	)
	if err != nil {
		fmt.Println("声明队列失败", err)
		return
	}

	// 2.从队列获取消息（消费者只关注队列）consume方式会不断的从队列中获取消息
	msgChanl, err := mq.Channel.Consume(
		mq.QueueName, // 队列名
		"",           // 消费者名，用来区分多个消费者，以实现公平分发或均等分发策略
		true,         // 是否自动应答
		false,        // 是否排他
		false,        // 是否接收只同一个连接中的消息，若为true，则只能接收别的conn中发送的消息
		true,         // 队列消费是否阻塞
		nil,          // 额外属性
	)
	if err != nil {
		fmt.Println("获取消息失败", err)
		return
	}

	for msg := range msgChanl {
		// 这里写你的处理逻辑
		// 获取到的消息是amqp.Delivery对象，从中可以获取消息信息
		fmt.Println(string(msg.Body))
		// msg.Ack(true) // 主动应答

	}

}
~~~

