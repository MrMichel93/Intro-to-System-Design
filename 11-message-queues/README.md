# Message Queues

## What are Message Queues?

**Message Queues** are systems that enable asynchronous communication between different parts of an application. Think of it like a post office - you drop off a letter (message) in a mailbox (queue), and it gets delivered to the recipient later.

**Key Concept**: Producer sends messages, Consumer receives messages, Queue stores messages in between.

```
[Producer] → [Message Queue] → [Consumer]
   |              |                |
 Creates       Stores           Processes
 message       message          message
```

## Why Use Message Queues?

### Without Message Queue (Synchronous):
```
User uploads video → Server processes video (takes 5 minutes) → User waits...
```
**Problem**: User must wait for long operation to complete!

### With Message Queue (Asynchronous):
```
User uploads video → Server: "Thanks! Processing..." (instant response)
                  → Queue stores job
                  → Worker processes video in background
                  → User gets notification when done
```
**Solution**: User doesn't wait, better experience!

## Benefits

1. **Decoupling**: Services don't need to know about each other
2. **Scalability**: Add more consumers to process messages faster
3. **Reliability**: Messages aren't lost if consumer is down
4. **Load Leveling**: Handle traffic spikes smoothly
5. **Asynchronous Processing**: Don't block users waiting

## Common Use Cases

### 1. Background Jobs
```
User submits report → Queue → Worker generates PDF → Email user
```

### 2. Email/SMS Notifications
```
User signs up → Queue → Worker sends welcome email
Order placed → Queue → Worker sends confirmation SMS
```

### 3. Image/Video Processing
```
Upload photo → Queue → Worker: resize, optimize, generate thumbnails
```

### 4. Event-Driven Architecture
```
Order created → Queue → Multiple consumers:
                         - Update inventory
                         - Send confirmation email
                         - Update analytics
```

## Popular Message Queue Systems

### 1. RabbitMQ
- Full-featured message broker
- Supports multiple messaging patterns
- Good for complex routing

### 2. Apache Kafka
- High-throughput distributed streaming platform
- Best for event streaming, logs
- Can handle millions of messages/second

### 3. AWS SQS (Simple Queue Service)
- Fully managed by AWS
- Easy to use
- Pay per use

### 4. Redis (with Pub/Sub or Streams)
- Fast, in-memory
- Simple pub/sub
- Good for real-time applications

## Message Queue Patterns

### 1. Point-to-Point (Work Queue)
One message → One consumer

```
Producer → [Queue] → Consumer 1
                  → Consumer 2 (competes for messages)
                  → Consumer 3
```

**Use case**: Distribute tasks among workers

### 2. Publish/Subscribe (Pub/Sub)
One message → Multiple consumers

```
Producer → [Topic] → Subscriber 1 (gets copy)
                  → Subscriber 2 (gets copy)
                  → Subscriber 3 (gets copy)
```

**Use case**: Broadcast events to multiple services

### 3. Request/Reply
Two-way communication

```
Client → [Request Queue] → Server
Client ← [Reply Queue] ← Server
```

## Key Concepts

### Message
Data sent through the queue

```json
{
  "id": "msg-123",
  "type": "order_created",
  "timestamp": "2024-01-15T10:30:00Z",
  "data": {
    "order_id": 456,
    "user_id": 789,
    "total": 99.99
  }
}
```

### Producer
Sends messages to the queue

```python
def create_order(order_data):
    # Save order to database
    order = db.save_order(order_data)
    
    # Send message to queue
    queue.send({
        "type": "order_created",
        "order_id": order.id,
        "user_id": order.user_id
    })
    
    return order
```

### Consumer
Receives and processes messages

```python
def process_order_message(message):
    order_id = message['order_id']
    
    # Send confirmation email
    send_email(order_id)
    
    # Update inventory
    update_inventory(order_id)
    
    # Acknowledge message processed
    message.ack()
```

### Acknowledgment (ACK)
Consumer tells queue: "I processed this message successfully"

- **ACK**: Message processed successfully, remove from queue
- **NACK**: Failed, requeue for retry
- **No ACK**: Consumer crashed, message returns to queue

## Handling Failures

### 1. Retry Mechanism
```python
def process_message_with_retry(message, max_retries=3):
    for attempt in range(max_retries):
        try:
            process_message(message)
            message.ack()
            return
        except TransientError:
            if attempt == max_retries - 1:
                # Max retries reached, send to dead letter queue
                send_to_dlq(message)
            else:
                # Wait and retry
                time.sleep(2 ** attempt)
```

### 2. Dead Letter Queue (DLQ)
Messages that fail repeatedly go to special queue for investigation

```
[Main Queue] → Consumer (fails 3 times) → [Dead Letter Queue]
                                             ↓
                                        Manual review
```

### 3. Idempotency
Make operations safe to repeat

```python
# BAD: Not idempotent
def process_payment(order_id):
    charge_credit_card(order_id)  # Charging twice if message reprocessed!

# GOOD: Idempotent
def process_payment(order_id):
    if not payment_exists(order_id):
        charge_credit_card(order_id)
```

## Best Practices

1. **Keep Messages Small**: Don't send large data, send references
2. **Make Consumers Idempotent**: Safe to process same message multiple times
3. **Set TTL (Time To Live)**: Messages expire if not processed
4. **Monitor Queue Depth**: Alert if queue growing (consumers too slow)
5. **Use Dead Letter Queues**: Don't lose failed messages
6. **Order Not Guaranteed**: Don't assume messages processed in order (unless using FIFO queue)
7. **Error Handling**: Always handle failures gracefully

## Real-World Examples

### Netflix
- Uses Kafka for event streaming
- Processes billions of events per day
- Tracks user viewing behavior

### Uber
- Message queues for dispatch system
- Matches riders with drivers
- Handles surge pricing

### Instagram
- Background job queues for image processing
- Resize images, generate thumbnails
- Apply filters

## RabbitMQ Example

```python
import pika

# Producer
connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()

# Declare queue
channel.queue_declare(queue='tasks')

# Send message
channel.basic_publish(
    exchange='',
    routing_key='tasks',
    body='Process video: video-123.mp4'
)

connection.close()

# Consumer
def callback(ch, method, properties, body):
    print(f"Processing: {body}")
    # Do work
    time.sleep(5)  # Simulate processing
    print("Done")
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='tasks', on_message_callback=callback)
channel.start_consuming()
```

## Summary

- Message queues enable asynchronous communication
- Decouple services and improve reliability
- Producers send messages, consumers process them
- Messages stored in queue until processed
- Support patterns: point-to-point, pub/sub
- Handle failures with retries and dead letter queues
- Make consumers idempotent
- Monitor queue depth and processing times

## Next Steps

Complete the challenges, then learn about **[CDN](../08-cdn/)** to understand how to deliver content faster globally!

## Implementation Approaches

### Approach 1: Simple Queue (AWS SQS, Azure Queue)
- **Description**: Managed queue service for basic message passing without broker complexity.
- **Pros**: Easy to setup, fully managed, auto-scaling, pay-per-use, no maintenance, built-in redundancy.
- **Cons**: Limited features (no routing), potential message duplication, limited ordering guarantees, vendor lock-in.
- **When to use**: Simple background jobs, event notifications, small to medium scale, cloud-native apps, want minimal ops overhead.

### Approach 2: Message Broker (RabbitMQ, ActiveMQ)
- **Description**: Full-featured message broker with routing, exchanges, and complex messaging patterns.
- **Pros**: Rich features (routing, pub/sub, RPC), message ordering, priorities, flexible, battle-tested, many client libraries.
- **Cons**: Need to manage/operate, cluster complexity, resource intensive, requires expertise, scaling requires planning.
- **When to use**: Complex routing needs, multiple messaging patterns, need message ordering, on-premise deployments, fine-grained control.

### Approach 3: Event Streaming (Kafka, Kinesis)
- **Description**: Distributed commit log for high-throughput event streaming and processing.
- **Pros**: Extremely high throughput, message persistence, replay capability, real-time processing, multiple consumers, built for scale.
- **Cons**: Higher complexity, overkill for simple use cases, operational overhead, requires Kafka expertise, storage costs.
- **When to use**: Event sourcing, log aggregation, real-time analytics, very high volume (millions/second), need message history/replay.

### Approach 4: In-Memory Queue (Redis, Memcached)
- **Description**: Fast in-memory data structures for low-latency queueing.
- **Pros**: Extremely fast (submillisecond), simple, low latency, easy to setup, good for rate limiting.
- **Cons**: Not durable (data loss on crash), limited persistence, not for critical messages, memory constraints.
- **When to use**: Rate limiting, temporary queues, very low latency needed, non-critical messages, cache-aside pattern.

## Trade-offs

| Aspect | Simple Queue | Message Broker | Event Stream |
|--------|--------------|----------------|--------------|
| Throughput | Medium (1K-10K/s) | Medium (10K-100K/s) | Very High (1M+/s) |
| Latency | 10-100ms | 1-10ms | 1-5ms |
| Ordering | Best effort | Guaranteed | Guaranteed (partition) |
| Persistence | Temporary | Durable | Long-term |
| Replay | No | No | Yes |
| Complexity | Low | Medium | High |
| Cost | Low | Medium | Higher |
| Operations | Minimal | Moderate | Significant |

| Aspect | At-Most-Once | At-Least-Once | Exactly-Once |
|--------|--------------|---------------|--------------|
| Delivery Guarantee | May lose messages | May duplicate | Single delivery |
| Performance | Fastest | Fast | Slower |
| Complexity | Simple | Moderate | Complex |
| Use Case | Non-critical logs | Most applications | Financial transactions |

## Capacity Calculations

### Queue Sizing
```
Message rate: 1,000 messages/second
Average message size: 10 KB
Processing time: 100ms per message

Queue depth calculation:
Messages in flight: 1,000 msg/s × 0.1s = 100 messages
Peak (3x): 300 messages

Consumer capacity needed:
1,000 msg/s ÷ 10 msg/s per consumer = 100 consumers

With buffer: 150 consumers

Storage requirements (1-day retention):
1,000 msg/s × 10 KB × 86,400s = 864 GB/day
```

### Throughput Planning
```
Business requirement: 10M orders/day
Peak hour (10% of daily): 1M orders/hour
Peak minute: 17K orders/minute = 280 orders/second

Message types:
- Order created: 280 msg/s
- Payment processing: 280 msg/s  
- Notification: 280 msg/s
- Analytics: 280 msg/s
Total: 1,120 msg/s

Queue system capacity:
RabbitMQ: 20K msg/s ✓
Kafka: 1M+ msg/s ✓✓
SQS: 3K msg/s per queue (need 1 queue) ✓
```

### Dead Letter Queue Monitoring
```
Main queue throughput: 10,000 msg/s
Expected error rate: 0.1%
DLQ rate: 10 msg/s

Alert thresholds:
Warning: DLQ rate > 50 msg/s (0.5% error)
Critical: DLQ rate > 200 msg/s (2% error)

Review DLQ daily for patterns
```

## Common Patterns

**Work Queue**: Single queue with multiple competing consumers. Each message processed by exactly one consumer. Load balanced across workers automatically.

**Pub/Sub**: Publisher sends to topic, multiple subscribers receive copies. Enables event broadcasting to multiple services. Good for notifications.

**Request-Reply**: Client sends request to queue, worker processes and replies to response queue. Enables asynchronous RPC. Use correlation ID to match responses.

**Priority Queue**: Higher priority messages processed first. VIP users or critical tasks get faster processing. Prevents head-of-line blocking.

**Delayed Queue**: Messages become visible after delay. Useful for retry with backoff, scheduled tasks, rate limiting. Native in SQS, plugin in RabbitMQ.

**Message Routing**: Route messages based on content/headers. Orders to order-queue, notifications to notification-queue. Keeps concerns separated.

**Circuit Breaker with Queue**: Stop publishing to queue when downstream service failing. Prevent message buildup. Resume when service healthy.

## Anti-Patterns (What NOT to Do)

**Synchronous Queue Polling**: Constantly polling empty queue wastes resources. Use long polling or push-based consumption instead.

**Large Messages**: Sending large payloads (images, files) through queue. Store in S3/blob storage, send reference in message. Keeps queue fast.

**No Error Handling**: Not handling consumer failures. Messages reprocessed infinitely or lost. Implement retries with exponential backoff and DLQ.

**Ignoring Queue Depth**: Not monitoring queue size. Growing queue indicates consumers can't keep up. Scale consumers or investigate bottleneck.

**No Idempotency**: Processing same message twice causes issues (double charge). Make consumers idempotent. Check if already processed before acting.

**Queue as Database**: Using queue for long-term storage. Queues are for transit, not storage. Messages should be processed and removed quickly.

**Fire and Forget**: Publishing without confirmation. Message might not reach queue. Wait for acknowledgment for critical messages.

**No DLQ Strategy**: Messages that fail repeatedly stay in queue forever. Configure DLQ after N retries. Review DLQ regularly.

## Interview Tips

**Explain Async Benefits**: "Message queues decouple services. Producer doesn't wait for consumer. Better scalability and failure isolation."

**Discuss Guarantees**: "At-least-once delivery means duplicates possible. Make consumers idempotent. Exactly-once is expensive, use only when needed."

**Calculate Capacity**: "With 1K msg/s and 100ms processing, need 100 consumers. Add 50% buffer for 150 consumers."

**Error Handling**: "Implement exponential backoff retries. After 3 attempts, send to DLQ for manual review. Log all failures."

**Queue vs Database**: "Queue for communication and task distribution. Database for persistent storage. Don't use queue as database."

**Real Examples**: "Netflix uses SQS for encoding jobs. Uber uses Kafka for trip events. LinkedIn uses Kafka for activity streams."

**Monitoring**: "Track queue depth, consumer lag, processing time, error rates, DLQ size. Alert on growing queue depth."

**Failure Scenarios**: "If consumer crashes, message returns to queue after visibility timeout. If queue service down, buffer in producer or circuit break."

## See It In Action

Complete the message queue challenges: **[Message Queue Challenges](./challenges.md)**

Design a message queue system for an e-commerce platform handling order processing at Black Friday scale.

## Additional Resources

**Documentation**:
- [AWS SQS Documentation](https://docs.aws.amazon.com/sqs/)
- [RabbitMQ Tutorial](https://www.rabbitmq.com/getstarted.html)
- [Apache Kafka Documentation](https://kafka.apache.org/documentation/)

**Articles**:
- [Microservices Messaging: Why REST Isn't Always the Best Choice](https://www.infoq.com/articles/messaging-rest-not-always-best/)
- [You Cannot Have Exactly-Once Delivery](https://bravenewgeek.com/you-cannot-have-exactly-once-delivery/)
- [Understanding When to Use RabbitMQ or Apache Kafka](https://content.pivotal.io/blog/understanding-when-to-use-rabbitmq-or-apache-kafka)

**Tools**:
- **RabbitMQ Management**: Web UI for queue monitoring
- **Kafka Manager**: Cluster management tool
- **Celery**: Distributed task queue for Python
- **Bull**: Redis-based queue for Node.js

**Books**:
- "Enterprise Integration Patterns" by Hohpe & Woolf
- "Kafka: The Definitive Guide" by Narkhede, Shapira, Palino
