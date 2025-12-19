# Design Exercise: Multi-Channel Notification System

## 1. Design Problem

### Problem Statement
Design a scalable notification system that can send messages through multiple channels (email, SMS, push notifications, in-app) to millions of users. The system must handle high throughput, ensure reliable delivery, support priority levels, respect user preferences, and provide delivery tracking and analytics.

### Context and Constraints
- Notifications triggered by various events (user actions, system events, scheduled)
- Multiple notification types (transactional, promotional, alerts)
- Different channels have different characteristics (email is slow but rich, SMS is fast but expensive)
- Users can opt-out of certain notification types
- Some notifications are time-sensitive (2FA codes, breaking news)
- Need to batch notifications to optimize costs
- Must handle delivery failures and retries
- Rate limits from external providers (Twilio, SendGrid)

### Requirements

#### Functional Requirements
- Send notifications via email, SMS, push notifications, in-app
- Support notification templates with personalization
- Prioritize notifications (critical, high, medium, low)
- Respect user preferences and opt-outs
- Schedule notifications for future delivery
- Track delivery status and failures
- Support batching for efficiency
- Handle multiple languages and timezones
- Provide delivery analytics and reporting

#### Non-Functional Requirements
- **Scale**: 100 million users, 1 billion notifications per day
- **Throughput**: Peak 50K notifications/second
- **Latency**: Critical notifications < 1 second, others < 1 minute
- **Reliability**: 99.9% delivery rate, retry failed deliveries
- **Cost**: Optimize provider costs through batching and routing
- **Compliance**: GDPR, CAN-SPAM, TCPA regulations

## 2. Guided Questions

### Understanding Message Queues
1. **Why use a message queue instead of sending notifications directly?**
   - Hint: What happens if notification service crashes?
   - Consider: Decoupling, reliability, scalability

2. **What happens if messages pile up faster than we can process them?**
   - Hint: Black Friday, breaking news, viral content
   - Consider: Queue overflow, back pressure, prioritization

### Queue Design
3. **Should we use one queue or multiple queues?**
   - Hint: Email vs. SMS vs. push have different characteristics
   - Consider: Priority levels, channel-specific processing

4. **How do we ensure messages aren't lost?**
   - Hint: Server crashes, network failures
   - Consider: Message persistence, acknowledgments, dead letter queues

### Processing Strategy
5. **How do we handle rate limits from providers?**
   - Hint: SendGrid allows 100 emails/second, Twilio charges per SMS
   - Consider: Throttling, batching, multiple providers

6. **What if a notification fails to send?**
   - Hint: Email server down, invalid phone number
   - Consider: Retry logic, exponential backoff, giving up

## 3. Step-by-Step Guidance

### Step 1: Choose Message Queue System
Select appropriate technology:

**Options:**
- **RabbitMQ**: Full-featured, good for complex routing
- **Apache Kafka**: High throughput, good for event streaming
- **AWS SQS**: Managed, simple, cost-effective
- **Redis**: Fast, in-memory, good for low latency

**For Notifications**: Use **Apache Kafka** or **AWS SQS**
- High throughput needed (1B notifications/day)
- Need message ordering for some cases
- Need message persistence
- Want managed solution (less operational overhead)

### Step 2: Design Queue Architecture
Multiple queues for different purposes:

```
Event Sources → [API Gateway] → [Topic: notifications]
                                        ↓
                        ┌───────────────┼───────────────┐
                        ↓               ↓               ↓
               [Queue: email]   [Queue: sms]    [Queue: push]
                        ↓               ↓               ↓
               [Email Workers]  [SMS Workers]  [Push Workers]
                        ↓               ↓               ↓
                  [SendGrid]      [Twilio]        [FCM/APNS]
```

**Queue Types:**
- `notifications.email.critical` - Time-sensitive emails
- `notifications.email.normal` - Regular emails
- `notifications.sms.critical` - OTP, alerts
- `notifications.sms.normal` - Reminders
- `notifications.push.critical` - Breaking news
- `notifications.push.normal` - General updates
- `notifications.inapp` - In-app notifications

### Step 3: Message Format
Standardized notification message:

```json
{
  "notification_id": "notif_abc123",
  "type": "order_confirmation",
  "priority": "high",
  "channel": "email",
  "recipient": {
    "user_id": "usr_456",
    "email": "user@example.com",
    "phone": "+1-555-0100",
    "language": "en",
    "timezone": "America/Los_Angeles"
  },
  "template": "order_confirmation_v1",
  "data": {
    "order_id": "ord_789",
    "total": 99.99,
    "items": [...]
  },
  "metadata": {
    "source": "order_service",
    "created_at": "2024-12-19T10:45:00Z",
    "expires_at": "2024-12-19T11:45:00Z"
  }
}
```

### Step 4: Implement Workers
Workers consume from queues and send notifications:

```python
import boto3
from kafka import KafkaConsumer

class EmailWorker:
    def __init__(self):
        self.sendgrid = SendGridClient(api_key=SENDGRID_KEY)
        self.consumer = KafkaConsumer(
            'notifications.email',
            bootstrap_servers=['kafka:9092'],
            group_id='email-workers',
            enable_auto_commit=False
        )
    
    def process_messages(self):
        for message in self.consumer:
            notification = json.loads(message.value)
            
            try:
                self.send_email(notification)
                # Commit offset only after successful send
                self.consumer.commit()
            except Exception as e:
                logger.error(f"Failed to send: {e}")
                # Move to dead letter queue
                self.dead_letter_queue.send(notification)
    
    def send_email(self, notification):
        template = self.get_template(notification['template'])
        rendered = template.render(notification['data'])
        
        self.sendgrid.send(
            to=notification['recipient']['email'],
            subject=rendered['subject'],
            body=rendered['body']
        )
```

### Step 5: Handle Failures and Retries
Robust error handling:

```python
class NotificationSender:
    MAX_RETRIES = 3
    RETRY_DELAYS = [60, 300, 900]  # 1min, 5min, 15min
    
    def send_with_retry(self, notification):
        for attempt in range(self.MAX_RETRIES):
            try:
                self.send(notification)
                self.update_status(notification['notification_id'], 'sent')
                return
            except TemporaryFailure as e:
                # Retryable error (rate limit, temporary outage)
                if attempt < self.MAX_RETRIES - 1:
                    delay = self.RETRY_DELAYS[attempt]
                    self.schedule_retry(notification, delay)
                    return
            except PermanentFailure as e:
                # Non-retryable error (invalid email, unsubscribed)
                self.update_status(notification['notification_id'], 'failed')
                return
        
        # Max retries exceeded
        self.move_to_dead_letter_queue(notification)
```

### Step 6: Implement Batching
Optimize costs by batching:

```python
class BatchedEmailSender:
    BATCH_SIZE = 100
    BATCH_TIMEOUT = 10  # seconds
    
    def __init__(self):
        self.batch = []
        self.last_send = time.time()
    
    def add_to_batch(self, notification):
        self.batch.append(notification)
        
        # Send if batch full or timeout reached
        if len(self.batch) >= self.BATCH_SIZE:
            self.send_batch()
        elif time.time() - self.last_send > self.BATCH_TIMEOUT:
            self.send_batch()
    
    def send_batch(self):
        if not self.batch:
            return
        
        # SendGrid batch API (cheaper, faster)
        self.sendgrid.send_batch([
            {
                'to': n['recipient']['email'],
                'template_id': n['template'],
                'data': n['data']
            }
            for n in self.batch
        ])
        
        self.batch = []
        self.last_send = time.time()
```

### Step 7: Rate Limiting
Respect provider limits:

```python
from ratelimit import limits, sleep_and_retry

class SMSSender:
    # Twilio allows 100 SMS/second
    @sleep_and_retry
    @limits(calls=100, period=1)
    def send_sms(self, phone, message):
        self.twilio.messages.create(
            to=phone,
            from_=TWILIO_PHONE,
            body=message
        )
```

## 4. Sample Solution

### Complete Architecture

```
┌─────────────────────────────────────────────┐
│         Event Sources                       │
│  - User Actions                             │
│  - System Events                            │
│  - Scheduled Jobs                           │
└──────────────┬──────────────────────────────┘
               ↓
┌──────────────▼──────────────────────────────┐
│      Notification API Service               │
│  - Validate request                         │
│  - Check user preferences                   │
│  - Enrich with user data                    │
│  - Route to appropriate queue               │
└──────────────┬──────────────────────────────┘
               ↓
┌──────────────▼──────────────────────────────┐
│         Apache Kafka / AWS SQS              │
│                                             │
│  Topics/Queues:                             │
│  - notifications.email.critical             │
│  - notifications.email.normal               │
│  - notifications.sms.critical               │
│  - notifications.sms.normal                 │
│  - notifications.push                       │
│  - notifications.inapp                      │
│  - notifications.dlq (dead letter)          │
└────┬────────┬────────┬────────┬─────────────┘
     │        │        │        │
     ↓        ↓        ↓        ↓
┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│ Email   ││  SMS    ││  Push   ││ In-App  │
│ Workers ││ Workers ││ Workers ││ Workers │
│ (x20)   ││ (x10)   ││ (x10)   ││ (x5)    │
└────┬────┘└────┬────┘└────┬────┘└────┬────┘
     │          │          │          │
     ↓          ↓          ↓          ↓
┌─────────┐┌─────────┐┌─────────┐┌─────────┐
│SendGrid ││ Twilio  ││   FCM   ││  Redis  │
│  API    ││   API   ││  APNS   ││         │
└─────────┘└─────────┘└─────────┘└─────────┘
     │          │          │          │
     └──────────┴──────────┴──────────┘
                    ↓
        ┌───────────────────────┐
        │   PostgreSQL DB       │
        │  - Notification logs  │
        │  - Delivery status    │
        │  - User preferences   │
        └───────────────────────┘
```

### Key Design Decisions

#### 1. Separate Queues by Channel
**Decision**: One queue per channel (email, SMS, push)

**Why**:
- Different processing characteristics
- Independent scaling (more email workers, fewer SMS)
- Channel-specific rate limiting
- Easier monitoring and debugging

#### 2. Priority Queues
**Decision**: Critical and normal queues for each channel

**Why**:
- 2FA codes must send immediately
- Marketing emails can wait
- Prevents low-priority from blocking high-priority

#### 3. At-Least-Once Delivery
**Decision**: Messages may be delivered multiple times

**Why**:
- Guaranteed delivery more important than exactly-once
- Simpler implementation
- Most notifications are idempotent anyway

**Trade-off**: Need idempotency keys to handle duplicates

## 5. Extension Challenges

### Challenge 1: Handle 10x Traffic Spike
**Scenario**: Viral event causes 10x notification volume

**How would you handle this?**
<details>
<summary>Solution</summary>

1. **Auto-Scaling Workers**:
   ```yaml
   # Kubernetes HPA
   apiVersion: autoscaling/v2
   kind: HorizontalPodAutoscaler
   metadata:
     name: email-worker
   spec:
     minReplicas: 10
     maxReplicas: 100
     metrics:
     - type: External
       external:
         metric:
           name: queue_depth
         target:
           type: AverageValue
           averageValue: "100"
   ```

2. **Priority Dropping**:
   ```python
   # Drop low-priority notifications during overload
   if queue_depth > CRITICAL_THRESHOLD:
       if notification['priority'] == 'low':
           logger.info("Dropping low-priority notification")
           return
   ```

3. **Graceful Degradation**:
   - Disable non-critical notifications
   - Increase batching (send 1000 at once instead of 100)
   - Use cheaper/faster channels (push instead of SMS)

4. **Back Pressure**:
   ```python
   # Slow down producers if queue too deep
   if queue_depth > WARNING_THRESHOLD:
       time.sleep(1)  # Throttle incoming requests
   ```

</details>

### Challenge 2: Cost Optimization
**Requirement**: Reduce notification costs by 50%

**What would you optimize?**
<details>
<summary>Solution</summary>

1. **Intelligent Channel Selection**:
   ```python
   # Use cheapest effective channel
   def choose_channel(notification, user_preferences):
       if notification['priority'] == 'critical':
           return 'sms'  # Most reliable, expensive
       
       if user_preferences['push_enabled']:
           return 'push'  # Free
       
       if user_preferences['email_enabled']:
           return 'email'  # Cheap ($0.0001 per email)
       
       return 'sms'  # Last resort
   ```

2. **Aggressive Batching**:
   ```python
   # Send 1000 emails at once (bulk discount)
   BATCH_SIZE = 1000
   BATCH_TIMEOUT = 60  # Wait up to 1 minute
   ```

3. **Deduplicate Notifications**:
   ```python
   # Don't send same notification multiple times
   recent_notifications = redis.get(f"recent:{user_id}")
   if notification['type'] in recent_notifications:
       logger.info("Skipping duplicate notification")
       return
   ```

4. **Provider Selection**:
   ```python
   # Use cheaper providers for non-critical
   if notification['priority'] != 'critical':
       provider = cheapest_available_provider()
   else:
       provider = most_reliable_provider()
   ```

5. **User Preferences**:
   ```python
   # Let users choose frequency
   if user_preferences['digest_mode']:
       # Send daily digest instead of real-time
       add_to_digest(user_id, notification)
   ```

</details>

### Challenge 3: Compliance (GDPR, CAN-SPAM)
**Requirement**: Ensure legal compliance

**How would you implement this?**
<details>
<summary>Solution</summary>

1. **Consent Management**:
   ```sql
   CREATE TABLE user_preferences (
       user_id INT PRIMARY KEY,
       email_marketing BOOLEAN DEFAULT FALSE,
       email_transactional BOOLEAN DEFAULT TRUE,
       sms_marketing BOOLEAN DEFAULT FALSE,
       sms_transactional BOOLEAN DEFAULT TRUE,
       push_enabled BOOLEAN DEFAULT TRUE,
       consent_date TIMESTAMP,
       consent_source VARCHAR(50)
   );
   ```

2. **Unsubscribe Links**:
   ```python
   # Add unsubscribe link to all marketing emails
   template = f"""
   {content}
   
   ---
   To unsubscribe: {UNSUBSCRIBE_URL}?token={signed_token}
   """
   ```

3. **Double Opt-In**:
   ```python
   def subscribe_user(email, notification_type):
       # Send confirmation email
       send_email(email, "Please confirm subscription")
       # Only enable after user clicks confirm link
   ```

4. **Audit Trail**:
   ```sql
   CREATE TABLE notification_log (
       id BIGSERIAL PRIMARY KEY,
       user_id INT,
       notification_type VARCHAR(50),
       channel VARCHAR(20),
       status VARCHAR(20),
       sent_at TIMESTAMP,
       consent_status BOOLEAN
   );
   ```

5. **Right to be Forgotten**:
   ```python
   def delete_user_data(user_id):
       # Delete all notification logs
       db.execute("DELETE FROM notification_log WHERE user_id = %s", user_id)
       # Anonymize instead of delete if needed for compliance
       db.execute("UPDATE notification_log SET user_id = NULL WHERE user_id = %s", user_id)
   ```

</details>

---

## Summary

This design exercise demonstrates how to build a scalable notification system by:
- Using message queues for reliability and decoupling
- Implementing multiple queues for different channels and priorities
- Handling failures with retries and dead letter queues
- Optimizing costs through batching and intelligent routing
- Scaling workers based on queue depth
- Respecting user preferences and legal requirements

**Key Takeaways**:
1. **Message queues enable reliable async processing**
2. **Separate queues by channel and priority**
3. **Implement robust retry logic with exponential backoff**
4. **Batch operations to optimize costs**
5. **Auto-scale workers based on queue depth**
6. **Always respect user preferences and legal requirements**
