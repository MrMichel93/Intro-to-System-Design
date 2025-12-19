# Module 8: Replication - Self-Check Quiz

**Time Limit:** 20 minutes  
**Total Questions:** 10  
**Passing Score:** 70%

---

## Multiple Choice Questions

### Question 1
What is the primary purpose of database replication?

A) To partition data  
B) To improve data availability and reliability  
C) To encrypt data  
D) To compress data  

---

### Question 2
In a master-slave replication setup, where do write operations go?

A) Any slave  
B) Only to the master  
C) Distributed across all replicas  
D) To the closest replica  

---

### Question 3
What is a key advantage of master-master (multi-master) replication?

A) Simpler conflict resolution  
B) Lower latency for writes in multiple regions  
C) Stronger consistency guarantees  
D) Reduced storage costs  

---

### Question 4
What is "replication lag"?

A) The time it takes to set up replication  
B) The delay between a write on the master and its appearance on replicas  
C) The time to switch to a replica during failover  
D) The network latency between data centers  

---

### Question 5
Which replication mode ensures that writes are committed on a replica before acknowledging to the client?

A) Asynchronous replication  
B) Synchronous replication  
C) Semi-synchronous replication  
D) Delayed replication  

---

## Short Answer Questions

### Question 6
Explain the trade-offs between synchronous and asynchronous replication. When would you choose each?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 7
What is a "split-brain" scenario in database replication, and how can it be prevented?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 8
Describe how read replicas can improve application performance. What are the potential consistency issues?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 9
In a master-master replication setup, how would you handle write conflicts (e.g., two users updating the same record simultaneously in different regions)?

**Your Answer:**
```
[Write your answer here]
```

---

### Question 10
Design a replication strategy for a global e-commerce platform that operates in 3 continents (North America, Europe, Asia). Consider read/write patterns, consistency requirements, and latency.

**Your Answer:**
```
[Write your answer here]
```

---

**Note:** Check your answers against the answer key in `answers.md` after completing the quiz.
