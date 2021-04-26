---
title: Kafka Consumer Loop 异常处理
---

不论用什么语言的 Kafka 客户端，consumer worker 的逻辑通常是一个 fetch loop，每次获取一条或者一批消息，然后在循环里处理信息，成功以后自动或者手动 commit offset。

比如使用 segmentio/kafka-go[1] Go 客户端的 consumer worker 逻辑如下：

```go
func (s *Server) collectionWorker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			s.collectionService.KafkaReader.Close()
			return
		default:
			m, err := s.collectionService.KafkaReader.FetchMessage(ctx)
			if err != nil {
				fmt.Printf("fetch messages error: %v\n", err)
				// 消息 fetch 失败，重新 fetch
				break
			}

			// 消息处理逻辑

			// 消息反序列化并构建实体 o
			...

			if err := s.collectionService.Save(o) {
				// 服务异常处理，重新 fetch 消息
				// 注意这里 break 的是 select 结构，不是外层 for 循环
				break
			}

			s.collectionService.KafkaReader.CommitMessages(ctx, m)
		}
	}
}
```

上面的逻辑有一个问题，在服务异常的时候会丢失信息。Kafka 客户端每次 fetch(poll) 一批(batch)消息，存在 buffer 里，当在 consumer 的逻辑里调用服务出现异常的时候（比如网络抖动或者数据库超时），下一次 fetch 消息不会重新获取到上一次处理失败的消息，不论这些消息有没有 committed。因此假如下一批 fetch 的新消息处理成功并 committed，那么之前处理失败的消息就相当于丢失[2]了。

当然，如果把异常处理的 `break` 换成 `return`，出现异常时这个 consumer worker 就退出了，那么重启 consumer worker 以后 fetch 到的消息还是未经 commit 的失败消息，这种情况下不会丢失消息，但每次出现异常都需要重启进程显然也不可取。

```go
if err := s.collectionService.Save(o) {
	// 服务异常处理，退出 consumer loop
	// 这样不会丢失消息，但需要重启或重新创建 consumer worker
	return
}
```

为了修复这个这个问题，我们可以加上重试逻辑。在某个消息处理出现异常的时候，要想既不阻塞后续消息处理，又不丢失这条消息，可以将处理异常的消息发送到一个 retry topic，在 retry topic 里重新消费这个消息。比如 Uber Engineering 的实践[3]里采用了多级 retry topic 的方式，每级 retry 消息的消费有不同的延时以避免消息风暴。这种方式稍显复杂。

通常，每个消息的处理逻辑都应该保证是幂等的，因此可以多次消费一条消息，这也是实现 Kafka "at least once" 模式的要求。在这种保证的前提下，重复消费一条消息不应产生异常，如果有异常，应该是服务或者网络故障，可能会自动恢复或者需要人工干预修复，这时即使消费下一条消息也同样会出错。因此我们可以一直重试消费失败的消息，直到故障得到修复消息处理成功，然后自动消费下一个消息。

下面的重试逻辑借助一个 buffered channel 循环消费失败消息直到成功：

```go
func (s *Server) collectionWorker(ctx context.Context) {
	ch := make(chan kafka.Message, 1)
	// 消息处理函数
	do := func(m kafka.Message) error {
		// 由消息反序列化得到实体
		err, o := unmarshal(m)
		if err != nil {
			// 如果出现反序列化错误，不重试该消息
			return nil
		}

		err = s.collectionService.Save(o)
		return err
	}

	for {
		select {
		case <-ctx.Done():
			s.collectionService.KafkaReader.Close()
			return
		case m := <-ch:
			// retry back-off
			time.Sleep(3 * time.Second)
			fmt.Printf("*** RETRY message at topic/partition/offset %v/%v/%v\n", m.Topic, m.Partition, m.Offset)
			err := do(m)
			if err != nil {
				ch <- m
				break
			}
			s.collectionService.KafkaReader.CommitMessages(ctx, m)
		default:
			m, err := s.collectionService.KafkaReader.FetchMessage(ctx)
			if err != nil {
				fmt.Printf("fetch messages error: %v\n", err)
				break
			}
			err = do(m)
			if err != nil {
				ch <- m
				break
			}
			s.collectionService.KafkaReader.CommitMessages(ctx, m)
		}
	}
}
```

在上面的逻辑中，当一个消息处理出现异常，会向 buffered channel（len=1) 发送处理失败的消息，并 break select 结构。因为 buffered channel 此时有了数据，因此下一次 select 将执行重试逻辑。如果重试失败，那么继续向 buffered channel 写入该消息并开始下一次重试。如果重试成功，那么 buffered channel 就是空的（已被 select/case 读取），下一次 select 将执行 `default:` case 里的代码，也就是正常消费逻辑。

可以在重试逻辑里加上合适的 back-off 策略，比如例子中是 sleep 3 秒。在消息处理函数 `do()` 中，可以控制哪些异常需要重试，哪些可以忽略，比如数据库连接错误肯定需要重试，而消息反序列化错误就应该忽略该消息，因为下一次重试该消息仍然不会成功。通过 `do()` 函数的返回值 error 是否为 nil 来控制是否重试消息。


[1]: https://github.com/segmentio/kafka-go
[2]: https://github.com/segmentio/kafka-go/issues/84
[3]: https://eng.uber.com/reliable-reprocessing/