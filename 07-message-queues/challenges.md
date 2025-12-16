# Message Queues - Challenges & Questions

## 🎯 Multiple Choice Questions

### Question 1
What is the main benefit of using message queues?

A) Faster database queries  
B) Asynchronous processing and decoupling of services  
C) Reduced code complexity  
D) Automatic bug fixing  

<details>
<summary>Answer</summary>

**B) Asynchronous processing and decoupling of services**

Message queues allow services to communicate asynchronously without waiting for each other, and they don't need to know about each other directly.
</details>

---

### Question 2
In a point-to-point queue pattern, what happens to a message?

A) All consumers receive a copy  
B) Only one consumer receives and processes it  
C) It's broadcast to everyone  
D) It's stored permanently  

<details>
<summary>Answer</summary>

**B) Only one consumer receives and processes it**

Point-to-point (work queue) means each message is delivered to exactly one consumer. Multiple consumers compete for messages.
</details>

---

### Question 3
What is a Dead Letter Queue (DLQ)?

A) A queue for deleted messages  
B) A queue for messages that failed processing multiple times  
C) A backup queue  
D) A test queue  

<details>
<summary>Answer</summary>

**B) A queue for messages that failed processing multiple times**

After a message fails repeatedly (e.g., 3 retries), it's moved to a DLQ for manual review and debugging.
</details>

---

### Question 4
Why is idempotency important for message consumers?

A) It makes processing faster  
B) It ensures safe reprocessing of duplicate messages  
C) It reduces queue size  
D) It prevents message loss  

<details>
<summary>Answer</summary>

**B) It ensures safe reprocessing of duplicate messages**

Messages might be delivered multiple times (network issues, failures). Idempotent operations produce the same result even if executed multiple times.
</details>

---

## 🧩 Scenario-Based Challenges

### Challenge 1: Design Email Notification System

Design a system using message queues to send emails when:
- User registers (welcome email)
- Order is placed (confirmation email)
- Password reset requested

**Requirements:**
- Handle 10,000 emails/hour
- Retry failed emails
- Track email status

<details>
<summary>Sample Solution</summary>

**Architecture:**

```
[Web App] → [Email Queue] → [Email Workers (3-5 instances)]
                                    ↓
                            [Email Service (SendGrid)]
                                    ↓
                          [Status Update Queue]
                                    ↓
                          [Status Tracker Service]
```

**Implementation:**

```python
# Producer (Web App)
def send_welcome_email(user_id, email):
    message = {
        "type": "welcome_email",
        "user_id": user_id,
        "to": email,
        "timestamp": datetime.now()
    }
    email_queue.send(message)

# Consumer (Email Worker)
def process_email_message(message):
    try:
        # Send email
        result = email_service.send(
            to=message['to'],
            template=message['type'],
            data=message.get('data', {})
        )
        
        # Track status
        status_queue.send({
            "message_id": message['id'],
            "status": "sent",
            "result": result
        })
        
        # Acknowledge
        message.ack()
        
    except TransientError as e:
        # Retry
        message.nack(requeue=True)
    
    except PermanentError as e:
        # Send to DLQ
        dlq.send(message)
        message.ack()
```

**Scaling:**
- Monitor queue depth
- If queue > 1000, add more workers
- Use auto-scaling based on queue size
</details>

---

### Challenge 2: Order Processing Pipeline

Design event-driven order processing:
1. User places order
2. Validate inventory
3. Process payment
4. Send confirmation
5. Update analytics

Use message queues to coordinate.

<details>
<summary>Sample Solution</summary>

**Event Flow:**

```
Order Service → [order.created] 
                      ↓
        ┌─────────────┼─────────────┐
        ↓             ↓             ↓
Inventory       Payment      Analytics
 Service        Service       Service
    ↓              ↓
[inventory.    [payment.
 reserved]      processed]
    ↓              ↓
    └──────┬───────┘
           ↓
   [order.confirmed]
           ↓
   Notification Service
```

**Implementation:**

```python
# Order Service
def create_order(order_data):
    order = db.save_order(order_data, status="pending")
    
    # Publish event
    publish("order.created", {
        "order_id": order.id,
        "user_id": order.user_id,
        "items": order.items,
        "total": order.total
    })
    
    return order

# Inventory Service
subscribe("order.created", on_order_created)

def on_order_created(event):
    if check_stock(event['items']):
        reserve_stock(event['items'])
        publish("inventory.reserved", event)
    else:
        publish("inventory.insufficient", event)

# Payment Service
subscribe("inventory.reserved", on_inventory_reserved)

def on_inventory_reserved(event):
    try:
        payment = process_payment(event)
        publish("payment.processed", {
            **event,
            "payment_id": payment.id
        })
    except PaymentError:
        publish("payment.failed", event)
        # Trigger compensating transaction
        publish("inventory.release", event)

# Order Service (listens for results)
subscribe("payment.processed", on_payment_processed)

def on_payment_processed(event):
    db.update_order_status(event['order_id'], "confirmed")
    publish("order.confirmed", event)

# Notification Service
subscribe("order.confirmed", send_confirmation_email)
```
</details>

---

## Next Steps

Great work! Now explore **[CDN](../08-cdn/)** to learn about delivering content faster globally!
