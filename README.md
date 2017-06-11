
# 分布式消息处理平台

## 这是**go-NSQ**的工程应用

- NSQ 是实时的分布式消息处理平台，其设计的目的是用来大规模地处理每天数以十亿计级别的消息。

- NSQ 具有分布式和去中心化拓扑结构，该结构具有无单点故障、故障容错、高可用性以及能够保证消息的可靠传递的特征。

- NSQ 非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。另外，官方还提供了拆箱即用 Go 和 Python 库。


## NSQ的自身特色很明显，最主要的优势在如下三个方面：

- 1，性能。在多个著名网站生产环境中被采用，每天能够处理数亿级别的消息。参见官方提供的性能说明文档

- 2，易用性。非常易于部署（几乎没有依赖）和配置（所有参数都可以通过命令行进行配置）。

- 3，可扩展性。具有分布式且无单点故障的拓扑结构，支持水平扩展，在无中断情况下能够无缝地添加集群节点。还具有强大的集群管理界面，参见nsqadmin

## NSQ分布式实时消息平台用法
NSQ是一个基于Go语言的分布式实时消息平台，可用于大规模系统中的实时消息服务，并且每天能够处理数亿级别的消息，其设计目标是为在分布式环境下运行的去中心化服务提供一个强大的基础架构。

NSQ非常容易配置和部署，且具有最大的灵活性，支持众多消息协议。

### NSQ是由四个重要组件构成：

- nsqd：一个负责接收、排队、转发消息到客户端的守护进程，它可以独立运行，不过通常它是由 nsqlookupd 实例所在集群配置的
- nsqlookupd：管理拓扑信息并提供最终一致性的发现服务的守护进程
- nsqadmin：一套Web用户界面，可实时查看集群的统计数据和执行各种各样的管理任务
- utilities：常见基础功能、数据流处理工具，如nsq_stat、nsq_tail、nsq_to_file、nsq_to_http、nsq_to_nsq、to_nsq

## 快速启动NSQ

```bash
brew install nsq
```

启动拓扑发现 

```bash
nsqlookupd
```

启动主服务、并注册 

```bash
nsqd --lookupd-tcp-address=127.0.0.1:4160
```

启动WEB UI管理程序 

```bash
nsqadmin --lookupd-http-address=127.0.0.1:4161
```

## 快速启动NSQ

brew install nsq

启动拓扑发现 nsqlookupd

启动主服务、并注册 nsqd --lookupd-tcp-address=127.0.0.1:4160

启动WEB UI管理程序 nsqadmin --lookupd-http-address=127.0.0.1:4161

## 简单使用演示

可以用浏览器访问* http://127.0.0.1:4171/ * 观察数据

也可尝试下 watch -n 0.5 "curl -s http://127.0.0.1:4151/stats" 监控统计数据

发布一个消息 

```bash
curl -d 'hello world 1' 'http://127.0.0.1:4151/put?topic=test'
```

创建一个消费者 

```bash
nsq_to_file --topic=test --output-dir=/tmp --lookupd-http-address=127.0.0.1:4161
```

Golang使用NSQ

```go
package main
import (
	"log"
	"time"
	"github.com/nsqio/go-nsq"
)
func main() {
	go startConsumer()
	startProducer()
}
// 生产者
func startProducer() {
	cfg := nsq.NewConfig()
	producer, err := nsq.NewProducer("127.0.0.1:4150", cfg)
	if err != nil {
		log.Fatal(err)
	}
	// 发布消息
	for {
		if err := producer.Publish("test", []byte("test message")); err != nil {
			log.Fatal("publish error: " + err.Error())
		}
		time.Sleep(1 * time.Second)
	}
}
// 消费者
func startConsumer() {
	cfg := nsq.NewConfig()
	consumer, err := nsq.NewConsumer("test", "sensor01", cfg)
	if err != nil {
		log.Fatal(err)
	}
	// 设置消息处理函数
	consumer.AddHandler(nsq.HandlerFunc(func(message *nsq.Message) error {
		log.Println(string(message.Body))
		return nil
	}))
	// 连接到单例nsqd
	if err := consumer.ConnectToNSQD("127.0.0.1:4150"); err != nil {
		log.Fatal(err)
	}
	<-consumer.StopChan
}
```