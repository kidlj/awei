---
title: Kafka Consumer Loop Error Handling - EN
---

Regardless of the language of the Kafka client, the logic of the consumer worker is usually a fetch loop that fetches one message or batch of messages at a time, processes the messages in the loop, and automatically or manually commits the offset when it succeeds.

For example, the consumer worker logic of a Go client using segmentio/kafka-go[1] is as follows:

```go
func (s *Service) Worker(ctx context.Context) {
	for {
		select {
		case <-ctx.Done():
			s.reader.Close()
			return
		default:
			m, err := s.reader.FetchMessage(ctx)
			if err != nil {
				fmt.Printf("fetch messages error: %v\n", err)
				// fetch failed，re-fetch message
				break
			}

			// Message proccessing logic

			// Message deserialization and construction of entities o
			...

			if err := s.Save(o) {
				// Service exception handling, re-fetch message
				// Note that the break here is the select structure, not the outer for loop
				break
			}

			s.reader.CommitMessages(ctx, m)
		}
	}
}
```

Even with the offset set for manual commit, the above logic still has a problem that would cause messages lost in case of a service exception. When the service (`s.Save(o)`) is called in the consumer loop logic and an exception occurs (e.g., network jitter or database timeout), the next fetch of messages does not reacquire the last failed message, whether it was committed or not. If the new message succeeds and is committed, then the previous failed message is lost[2].

Of course, if `break` is replaced by `return`, and the consumer worker exits when an exception occurs, the message fetched after restarting the consumer worker will still be the uncommitted failed message, in which case the message will not be lost, but it is obviously undesirable to restart the process every time an exception occurs

To fix this problem, we can add retry logic. When a message processing exception occurs, to not block the subsequent message processing and not lose the message, we can send the failed message to a retry topic and re-consume the message in the retry topic. For example, Uber Engineering's practice[3] uses a multi-level retry topic approach, where each level of retry has a different delay to avoid message storms. This approach is kind of complex to use.

In general, the processing logic of each message should be guaranteed to be idempotent, so that the same message can be consumed multiple times, which is a requirement to achieve the Kafka "at least once" guarantee. With this guarantee, repeated consumption of a message should not generate an exception, and if there is an exception, it should be a service or network failure that would automatically recover. So we can keep retrying to consume the failed message until the failure is fixed automatically or maunully, and then the next message is consumed.

The following retry logic uses a buffered channel to cycle through the failed messages until they succeed.

```go
func (s *Service) Worker(ctx context.Context) {
	// retry channel
	ch := make(chan kafka.Message, 1)
	// worker function
	do := func(m kafka.Message) error {
		// Deserialize from messages to get entities
		err, o := unmarshal(m)
		if err != nil {
			// If a deserialization error occurs, do not retry the message
			// Logging logic or persistence logic can be added here
			return nil
		}

		err = s.Save(o)
		return err
	}

	for {
		select {
		case <-ctx.Done():
			s.reader.Close()
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
			s.reader.CommitMessages(ctx, m)
		default:
			m, err := s.reader.FetchMessage(ctx)
			if err != nil {
				fmt.Printf("fetch messages error: %v\n", err)
				break
			}
			err = do(m)
			if err != nil {
				ch <- m
				break
			}
			s.reader.CommitMessages(ctx, m)
		}
	}
}
```
In the above logic, when a message processing exception occurs, the failed message is sent to a buffered channel (len=1) and the `select` structure breaks. Since the buffered channel has data at this point, the next select will perform the retry logic. If the retry fails, then the message continues to be written to the buffered channel and the next retry begins. If the retry succeeds, then the buffered channel is empty (read by the select/case) and the next select executes the code in the `default:` case, which is the normal consumption logic.

You can add an appropriate back-off policy to the retry logic, such as sleep for 3 seconds in the example. In the message processing function `do()`, you can control which exceptions should be retried and which can be ignored, for example, a database connection error should definitely be retried, while a message deserialization error should be ignored because the next retry of the message will still not succeed. Whether or not to retry a message is controlled by whether or not the return error value of the `do()` function is nil.

Further, if there are multiple similar message consumption logics in your program, the loop logic can be abstracted out into a generic helper function that takes a message processing function and its corresponding kafka reader as arguments.

```go
const RETRY_BACKOFF = 3 * time.Second

// generic helper loop function
func WithWorker(ctx context.Context, reader *kafka.Reader, do func(context.Context, kafka.Message) error) {
	// retry channel
	ch := make(chan kafka.Message, 1)

	for {
		select {
		case <-ctx.Done():
			reader.Close()
			return
		case m := <-ch:
			// retry back-off
			time.Sleep(RETRY_BACKOFF)
			fmt.Printf("*** RETRY message at topic/partition/offset %v/%v/%v\n", m.Topic, m.Partition, m.Offset)
			err := do(m)
			if err != nil {
				ch <- m
				break
			}
			reader.CommitMessages(ctx, m)
		default:
			m, err := reader.FetchMessage(ctx)
			if err != nil {
				fmt.Printf("fetch messages error: %v\n", err)
				break
			}
			err = do(m)
			if err != nil {
				ch <- m
				break
			}
			reader.CommitMessages(ctx, m)
		}
	}
}

// message processing function
func (s *Service) worker(ctx context.Context, m kafka.Message) error {
	// Deserialize from messages to get entities
	err, o := unmarshal(m)
	if err != nil {
		// If a deserialization error occurs, do not retry the message
		// Logging logic or persistence logic can be added here
		return nil
	}

	err = s.Save(ctx, o)
	return err
}
```

And then call it like this:

```go
func (s *Service) Run(ctx context.Context) {
	go utils.WithWorker(ctx, s.cacheReader, s.cacheWorker)
	go utils.WithWorker(ctx, s.indexReader, s.indexWorker)
}
```

[1]: https://github.com/segmentio/kafka-go
[2]: https://github.com/segmentio/kafka-go/issues/84
[3]: https://eng.uber.com/reliable-reprocessing/