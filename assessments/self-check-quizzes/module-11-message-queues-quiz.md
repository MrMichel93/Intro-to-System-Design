# Module 11: Message Queues - Self-Check Quiz

**Time Limit:** 20 minutes  
**Total Questions:** 10  
**Passing Score:** 70%

---

## Multiple Choice Questions

### Question 1
What is the primary benefit of using message queues in a distributed system?

A) Faster database queries  
B) Decoupling components and enabling asynchronous processing  
C) Reducing storage costs  
D) Improving security  

---

### Question 2
In a message queue system, what happens if the consumer is temporarily down?

A) Messages are lost  
B) Messages remain in the queue until the consumer recovers  
C) The producer must retry immediately  
D) The system crashes  

---

### Question 3
Which message delivery guarantee ensures a message is processed at least once, possibly multiple times?

A) At-most-once  
B) Exactly-once  
C) At-least-once  
D) Never-once  

---

### Question 4
Which popular message queue system is known for high throughput and is used for event streaming?

A) RabbitMQ  
B) Apache Kafka  
C) Amazon SQS  
D) Redis  

---

### Question 5
What is a "dead letter queue"?

A) A queue that is no longer in use  
B) A queue for storing messages that failed processing after multiple attempts  
C) A queue with low priority  
D) A queue that encrypts messages  

---

## Short Answer Questions

### Question 6
Explain the difference between a message queue and a message broker. What additional features does a message broker provide?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 7
Compare the push model (pub/sub) with the pull model (queue) for message distribution. When would you use each?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 8
What is the "poison message" problem in message queues, and how can it be handled?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 9
Describe how message queues enable better fault tolerance in a system. What happens when a processing node fails mid-operation?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 10
Design a message queue architecture for an e-commerce order processing system. Include order placement, payment processing, inventory update, and shipping notification. Explain the flow and why message queues are beneficial here.

**Your Answer:**
```
[Write your answer here]
```

---

**Note:** Check your answers against the answer key in `answers.md` after completing the quiz.
