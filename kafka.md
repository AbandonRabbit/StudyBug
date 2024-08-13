# 基础

**broker：** 理论上一个一台机器就是一个 broker，也可以是一台机器上有多个 broker 进程，不过要使用不同的端口号区分

**Topic：**一个broker内部有多个 topic ，存储某一类的业务数据

**Partition：**同一个 Topic 分成多个 Partition ，每一个 Partition 都是一个小的队列，同一个 Topic 的 同一个 Partition 又有很多备份，分为 Leader(主) 和从 Follower(从) 

**Leader 和 Follower ：**同一个 Topic 的 同一个 Partition 下的 Leader 和 Follower 存储在不同的 Broker上，主要容灾，当一个Broker挂掉，不至于数据丢失 

> - 把一个 Topic 分成多个 Partition 主要是为了方便消费的受负载均衡，一般情况下一个消费者或消费者组至少对应一个 Partition ，如果消费者或消费者数量大于 Partition ，则至少会有一个 消费者 处于阻塞状态

**offset：**在消费一个 Partition 里面的数据时，一边读取，一边向 kafka 集群汇报当前读到的位置，这个位置在 kafka 内部使用 offset 标记，每一个写进 kafka 中的数据都会有一个 offset 

# 实现过程

**生产者写入数据时怎么选择 Partition？**

1. 可以指定选择那个 Partition
2. 如果没有指定，会根据写入这条数据的 key 决定选择那个 Partition
3. 如果没有 key 就根据时间片轮询的方式选择

**确定 Partition 之后的工作流程：**

1. 首先询问 kafka 集群这个 Topic 的这个 Partition 的 leader 是那台服务器，是哪一个 broker 
2. 将数据发送给 leader 
3. leader 将数据写入本地磁盘
4. follower 从 leader 拉取消息数据
5. follower 将消息写入本地磁盘后向 leader 发送一个确认消息 ack
6. leader 向 生产者 发送一个确认消息 ack 

> 生产者可以设置是否需要对方返回 ack 消息
>
> - 0 不需要返回，安全性最低但是效 率最高。
> - 1 只确保leader发送成功。
> - all 完成所有 ack 确认流程，安全性最⾼高，但是效率最低。

# 代码示例

**消费者信息**

~~~go
func readKafka(ctx context.Context) {
	reader = kafka.NewReader(kafka.ReaderConfig{
		Brokers:        []string{"localhost:9092"}, //可以配置多个
		Topic:          topic,
		CommitInterval: 1 * time.Second,   //设置报告策略
		GroupID:        "rec_team",        //所属的消费者组
		StartOffset:    kafka.FirstOffset, //设置新的消费方消费数据起始位置。只在消费方创建时有效
	})
	//defer reader.Close()
	for {
		if msg, err := reader.ReadMessage(ctx); err != nil {
			fmt.Printf("读取kafka失败 %v\n", err)
			break
		} else {
			fmt.Printf(
				"topic:%s\t partition:%d\t offset:%d\t key:%s value:%s\t \n",
				msg.Topic,
				msg.Partition,
				msg.Offset,
				string(msg.Key),
				string(msg.Value),
			)
		}
	}
}
~~~

**生产者**

~~~go
func writeKafka(ctx context.Context) {
	writer := &kafka.Writer{
		Addr:                   kafka.TCP("localhost:9092"), //链接地址
		Topic:                  topic,                       //设置消息主题
		Balancer:               &kafka.Hash{},               //负载均衡算法
		WriteTimeout:           time.Second * 1,             //设置超时时间
		RequiredAcks:           kafka.RequireNone,           //设置超时确认级别
		AllowAutoTopicCreation: true,                        //是否可以自动创建 topic，一般为false，由运维创建
	}
	defer writer.Close()
	for i := 0; i < 3; i++ {
		if err := writer.WriteMessages( //批量写入，这个操作是原子操作
			ctx,
			kafka.Message{Key: []byte("1"), Value: []byte("h1")},
			kafka.Message{Key: []byte("2"), Value: []byte("h2")},
			kafka.Message{Key: []byte("3"), Value: []byte("h3")},
			kafka.Message{Key: []byte("4"), Value: []byte("h4")},
			kafka.Message{Key: []byte("5"), Value: []byte("h5")},
		); err != nil {
			if err == kafka.LeaderNotAvailable {
				time.Sleep(time.Millisecond * 500)
				continue
			} else {
				fmt.Printf("批量插入数据失败 kafka：%v\n", err)
			}
		} else {
			break
		}
	}
}
~~~

