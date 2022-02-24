---
title: MAC上RabbitMQ从安装到用GO快速实现搬移
categories: 开发
keywords: RabbitMQ
top_img: /img/4797861.webp
cover: /img/7anewqv0cvxa.webp
abbrlink: 5fe856cc
date: 2022-01-31 22:17:16
---

##  前言

最近在跟慕课做一个秒杀商城的小项目，接触了RabbitMQ

虽然平时是在Python中实现消息队列，但是不得不说RabbitMQ香呀

今天也是除夕，在这个祝大家**新年快乐**，发个小水文章吧QAQ

##  安装开始

```bash
# 基础安装
$ brew install rabbitmq
$ vim ~/.zshrc # 将 export PATH=$PATH:/usr/local/sbin 写入
$ rabbitmq-server # 重新打开终端，开启服务

# 开启插件
$ rabbitmq-plugins list # 查看插件
$ rabbitmq-plugins enable rabbitmq_management # 启动管理插件
$ rabbitmq-plugins enable rabbitmq_tracing # 启动日志
$ rabbitmq-plugins disable rabbitmq_tracing # 关闭日志

# 额外命令
$ rabbitmq-server -detached # 后台启动
$ rabbitmqctl status # 查看状态
$ rabbitmqctl stop # 关闭
```

## 打好基础

> 这些都是很基本的概念，你得明白什么是什么就好了
>
> 因为作为工具，首先要会用起来，会采用囫囵吞枣的模式学习
>
> 随着后面的深入，慢慢了解特点吧

|    概念     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|   Channel   | 生产者publish或是消费者subscribe一个队列都是通过信道来通信的 |
|  Exchange   | exchange的作用就是类似路由器，服务器会根据路由键将消息从交换器路由到队列上去 |
|    Queue    |                 队列收到的消息将发送给消费者                 |
|   Binding   |                    建立链接交换的绑定信息                    |
| VirtualHost |                    不同的隔离区，防止污染                    |
| Connection  |                          建立的链接                          |

---

|     工作模式      |                           描述                           |
| :---------------: | :------------------------------------------------------: |
|      simple       |                     最简单的收发模式                     |
|       work        |                        资源的竞争                        |
| publish/subscribe |                         共享资源                         |
|      routing      | 只能匹配上路由key对应的消息队列,对应的消费者才能消费消息 |
|       topic       |                  routing的一种模糊匹配                   |

##  代码实现

### 功能仓库文件

```go
// MQURL 连接信息 amqp://账号:密码@ip:host/vhost
const MQURL = "amqp://guest:guest@127.0.0.1:5672/"

// RabbitMQ rabbitMQ结构体
type RabbitMQ struct {
	conn      *amqp.Connection // 链接
	channel   *amqp.Channel    // 通道
	QueueName string           //队列名称
	Exchange  string           //交换机名称
	Key       string           //bind Key 名称
	Mqurl     string           //连接信息
}

// NewRabbitMQ 创建结构体实例
func NewRabbitMQ(queueName string, exchange string, key string) *RabbitMQ {
	return &RabbitMQ{QueueName: queueName, Exchange: exchange, Key: key, Mqurl: MQURL}
}

// Destroy 断开 channel 和 connection
func (r *RabbitMQ) Destroy() {
	r.channel.Close() // 断开 channel
	r.conn.Close()    // 断开 conn
}

// 错误处理函数
func (r *RabbitMQ) failOnErr(err error, message string) {
	if err != nil {
		log.Printf("%s:%s", message, err)         // 打印错误
		panic(fmt.Sprintf("%s:%s", message, err)) // 抛出错误
	}
}

// NewRabbitMQSimple 创建简单模式下RabbitMQ实例
// 在Simple模式下唯一不同的是 queueName
func NewRabbitMQSimple(queueName string) *RabbitMQ {
	// todo 创建RabbitMQ实例
	rabbitmq := NewRabbitMQ(queueName, "", "")
	var err error
	// todo 补上conn与channel
	//获取connection
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "failed to connect rabbitmq!")
	//获取channel
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "failed to open a channel")
	return rabbitmq
}

// PublishSimple 简单模式下队列生产
func (r *RabbitMQ) PublishSimple(message string) {
	// todo 申请队列，如果队列不存在会自动创建，存在则跳过创建
	_, err := r.channel.QueueDeclare(
		r.QueueName, // 首先放入名称
		false,       //是否持久化
		false,       //是否自动删除
		false,       //是否具有排他性
		false,       //是否阻塞处理
		nil,         //额外的属性
	)
	if err != nil {
		fmt.Println(err)
	}
	//todo 调用channel 发送消息到队列中
	r.channel.Publish(
		r.Exchange, // 此处为空
		r.QueueName,
		false, //如果为true，根据自身exchange类型和routeKey规则；无法找到符合条件的队列会把消息返还给发送者
		false, //如果为true，当exchange发送消息到队列后发现队列上没有消费者，则会把消息返还给发送者
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		})
}

// ConsumeSimple 简单模式下消费者
func (r *RabbitMQ) ConsumeSimple() {
	//todo 申请队列，如果队列不存在会自动创建，存在则跳过创建
	q, err := r.channel.QueueDeclare(
		r.QueueName,
		false, //是否持久化
		false, //是否自动删除
		false, //是否具有排他性
		false, //是否阻塞处理
		nil,   //额外的属性
	)
	if err != nil {
		fmt.Println(err)
	}

	//todo 接收消息
	msg, err := r.channel.Consume(
		q.Name, // queue
		"",     //用来区分多个消费者 此处不区分
		true,   //是否自动应答
		false,  //是否独有
		false,  //设置为true，表示不能将同一个Connection中生产者发送的消息传递给这个Connection中的消费者
		false,  // 是否阻塞处理
		nil,    // 额外的属性
	)
	if err != nil {
		fmt.Println(err)
	}

	//todo 启用协程处理消息

	// 此处使用forever的意思为因为协程会始终监听消息(除非手动结束)
	// 手动结束才会进行 <-forever 有协程且一直尝试读取数据
	forever := make(chan bool)
	go func() {
		for d := range msg {
			// 消息逻辑处理
			log.Printf("Received a message: %s", d.Body)
		}
	}()
	log.Printf(" [*] Waiting for messages. To exit press CTRL+C")
	<-forever
}

// NewRabbitMQPubSub 订阅模式创建RabbitMQ实例就要设置路由器了
func NewRabbitMQPubSub(exchangeName string) *RabbitMQ {
	//todo 创建RabbitMQ实例
	rabbitmq := NewRabbitMQ("", exchangeName, "")
	var err error
	//todo 获取connection和获取channel
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "failed to connect rabbitmq!")
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "failed to open a channel")
	return rabbitmq
}

// PublishPub 订阅模式生产
func (r *RabbitMQ) PublishPub(message string) {
	//todo 尝试创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		"fanout",
		true,
		false,
		false, //true表示这个exchange不可以被client用来推送消息，仅用来进行exchange和exchange之间的绑定
		false,
		nil,
	)

	r.failOnErr(err, "Failed to declare an exchange")

	//todo 发送消息
	err = r.channel.Publish(
		r.Exchange,
		"",
		false,
		false,
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		})
}

// ReceiveSub 订阅模式消费端代码
func (r *RabbitMQ) ReceiveSub() {
	//todo 试探性创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		"fanout", //交换机类型
		true,
		false,
		false, //true表示这个exchange不可以被client用来推送消息，仅用来进行exchange和exchange之间的绑定
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare an exchange")
	//todo 试探性创建队列，这里注意队列名称不要写
	q, err := r.channel.QueueDeclare(
		"", //随机生产队列名称
		false,
		false,
		true,
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare a queue")

	//todo 绑定队列到 exchange 中
	err = r.channel.QueueBind(
		q.Name,
		"", //在pub/sub模式下，这里的key要为空
		r.Exchange,
		false,
		nil)

	//todo 消费消息
	message, err := r.channel.Consume(
		q.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)
	forever := make(chan bool)
	go func() {
		for d := range message {
			log.Printf("Received a message: %s", d.Body)
		}
	}()
	fmt.Println("退出请按 CTRL+C\n")
	<-forever
}

// NewRabbitMQRouting 路由模式
func NewRabbitMQRouting(exchangeName string, routingKey string) *RabbitMQ {
	//todo 创建RabbitMQ实例
	rabbitmq := NewRabbitMQ("", exchangeName, routingKey)
	var err error
	//todo 获取connection 获取channel
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "failed to connect rabbitmq!")
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "failed to open a channel")
	return rabbitmq
}

// PublishRouting 路由模式发送消息
func (r *RabbitMQ) PublishRouting(message string) {
	//1.尝试创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		"direct",
		true,
		false,
		false,
		false,
		nil,
	)

	r.failOnErr(err, "Failed to declare an exchange")

	//2.发送消息
	err = r.channel.Publish(
		r.Exchange,
		//要设置
		r.Key,
		false,
		false,
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		})
}

// ReceiveRouting 路由模式接受消息
func (r *RabbitMQ) ReceiveRouting() {
	// todo 试探性创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		//交换机类型
		"direct",
		true,
		false,
		false,
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare an exchange")
	// todo 试探性创建队列，这里注意队列名称不要写
	q, err := r.channel.QueueDeclare(
		"", //随机生产队列名称
		false,
		false,
		true,
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare a queue")

	//绑定队列到 exchange 中
	err = r.channel.QueueBind(
		q.Name,
		//需要绑定key
		r.Key,
		r.Exchange,
		false,
		nil)

	// todo 消费消息
	message, err := r.channel.Consume(
		q.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)
	forever := make(chan bool)
	go func() {
		for d := range message {
			log.Printf("Received a message: %s", d.Body)
		}
	}()
	fmt.Println("退出请按 CTRL+C\n")
	<-forever
}

// NewRabbitMQTopic 话题模式
func NewRabbitMQTopic(exchangeName string, routingKey string) *RabbitMQ {
	// todo 创建RabbitMQ实例
	rabbitmq := NewRabbitMQ("", exchangeName, routingKey)
	var err error
	// todo 获取connection与获取channel
	rabbitmq.conn, err = amqp.Dial(rabbitmq.Mqurl)
	rabbitmq.failOnErr(err, "failed to connect rabbitmq!")
	rabbitmq.channel, err = rabbitmq.conn.Channel()
	rabbitmq.failOnErr(err, "failed to open a channel")
	return rabbitmq
}

// PublishTopic 话题模式发送消息
func (r *RabbitMQ) PublishTopic(message string) {
	// todo 尝试创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		"topic",
		true,
		false,
		false,
		false,
		nil,
	)

	r.failOnErr(err, "Failed to declare an exchange")

	// todo 发送消息
	err = r.channel.Publish(
		r.Exchange,
		//要设置
		r.Key,
		false,
		false,
		amqp.Publishing{
			ContentType: "text/plain",
			Body:        []byte(message),
		})
}

// ReceiveTopic 话题模式接受消息
//要注意key,规则
//其中“*”用于匹配一个单词，“#”用于匹配多个单词（可以是零个）
//匹配 xx.* 表示匹配 xx.hello, 但是 xx.hello.one需要用 xx.#才能匹配到
func (r *RabbitMQ) ReceiveTopic() {
	//1.试探性创建交换机
	err := r.channel.ExchangeDeclare(
		r.Exchange,
		//交换机类型
		"topic",
		true,
		false,
		false,
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare an exch"+
		"ange")
	//2.试探性创建队列，这里注意队列名称不要写
	q, err := r.channel.QueueDeclare(
		"", //随机生产队列名称
		false,
		false,
		true,
		false,
		nil,
	)
	r.failOnErr(err, "Failed to declare a queue")

	//绑定队列到 exchange 中
	err = r.channel.QueueBind(
		q.Name,
		//在pub/sub模式下，这里的key要为空
		r.Key,
		r.Exchange,
		false,
		nil)

	//消费消息
	message, err := r.channel.Consume(
		q.Name,
		"",
		true,
		false,
		false,
		false,
		nil,
	)

	forever := make(chan bool)

	go func() {
		for d := range message {
			log.Printf("Received a message: %s", d.Body)
		}
	}()

	fmt.Println("退出请按 CTRL+C\n")
	<-forever
}
```

###  各个模式实现

> 因为不确定包的位置
>
> 所以报红简单写一下引入上面的仓库文件就好了

1. 简单模式

```go
// 发布者
func main() {
	// todo 创建实例并发送消息
	rabbitmq := RabbitMQ.NewRabbitMQSimple("Simple")
	rabbitmq.PublishSimple("Hello world!")
	fmt.Println("发送成功！")
}
// 接受者
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQSimple("Simple") // 名字要一样
	rabbitmq.ConsumeSimple()
}
```

2. 工作模式

```go
// 发布者
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQSimple("Simple")
	for i := 0; i <= 100; i++ {
		rabbitmq.PublishSimple("Hello world!" + strconv.Itoa(i))
		time.Sleep(1 * time.Second)
		fmt.Println(i)
	}
}
// 接受者1
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQSimple("Simple")
	rabbitmq.ConsumeSimple()
}
// 接受者2
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQSimple("Simple")
	rabbitmq.ConsumeSimple()
}
```

3. 发布模式

```go
// 发布者
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQPubSub("newProduct")
	for i := 0; i < 100; i++ {
		rabbitmq.PublishPub("订阅模式生产第" + strconv.Itoa(i) + "条" + "数据")
		fmt.Println("订阅模式生产第" + strconv.Itoa(i) + "条" + "数据")
		time.Sleep(1 * time.Second)
	}
}
// 订阅者1
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQPubSub("newProduct")
	rabbitmq.ReceiveSub()
}
// 订阅者2
func main() {
	rabbitmq := RabbitMQ.NewRabbitMQPubSub("newProduct")
	rabbitmq.ReceiveSub()
}
```

4. 路由模式

```go
// 发布者
func main() {
	One := RabbitMQ.NewRabbitMQRouting("ex", "one")
	Two := RabbitMQ.NewRabbitMQRouting("ex", "two")
	for i := 0; i <= 10; i++ {
		One.PublishRouting("Hello one!" + strconv.Itoa(i))
		Two.PublishRouting("Hello Two!" + strconv.Itoa(i))
		time.Sleep(1 * time.Second)
		fmt.Println(i)
	}
}
// 接受者
func main() {
	One := RabbitMQ.NewRabbitMQRouting("ex", "one")
	One.ReceiveRouting()
}
// 接受者
func main() {
	Two := RabbitMQ.NewRabbitMQRouting("ex", "two")
	Two.ReceiveRouting()
}
```

5. 话题模式

```go
// 发布者
func main() {
	One := RabbitMQ.NewRabbitMQTopic("exTopic", "topic.one")
	Two := RabbitMQ.NewRabbitMQTopic("exTopic", "topic.two")
	for i := 0; i <= 10; i++ {
		One.PublishTopic("Hello topic one!" + strconv.Itoa(i))
		Two.PublishTopic("Hello topic Two!" + strconv.Itoa(i))
		time.Sleep(1 * time.Second)
		fmt.Println(i)
	}
}
// 接受者1
func main() {
	One := RabbitMQ.NewRabbitMQTopic("exTopic", "#") // # 表示一个或者多个词语
	One.ReceiveTopic()
}
// 接受者2
func main() {
	Two := RabbitMQ.NewRabbitMQTopic("exTopic", "*.two") // 表示多个词语
	Two.ReceiveTopic()
}
```