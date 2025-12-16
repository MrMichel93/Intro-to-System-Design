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
