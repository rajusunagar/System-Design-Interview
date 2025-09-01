# Message Queues

## Definition
Message queues enable asynchronous communication between services by storing messages in a queue until they can be processed.

## Benefits
- **Decoupling**: Services don't need to know about each other
- **Scalability**: Handle traffic spikes by queuing requests
- **Reliability**: Messages persist until processed
- **Fault Tolerance**: System continues working if components fail

## Message Queue Patterns

### Point-to-Point (Queue)
```
Producer → Queue → Consumer
```
- One message consumed by exactly one consumer
- **Use Cases**: Task processing, job queues

### Publish-Subscribe (Topic)
```
Publisher → Topic → Subscriber 1
                 → Subscriber 2
                 → Subscriber 3
```
- One message delivered to multiple subscribers
- **Use Cases**: Event notifications, broadcasting

### Request-Reply
```
Client → Request Queue → Server
Client ← Reply Queue ← Server
```
- Synchronous-like communication over async messaging
- **Use Cases**: RPC over messaging

## Message Delivery Guarantees

### At-Most-Once
- Messages delivered zero or one time
- **Risk**: Message loss possible
- **Benefit**: No duplicates, high performance

### At-Least-Once
- Messages delivered one or more times
- **Risk**: Duplicate messages possible
- **Benefit**: No message loss

### Exactly-Once
- Messages delivered exactly one time
- **Benefit**: No loss, no duplicates
- **Cost**: Complex implementation, performance overhead

## Message Ordering

### FIFO (First In, First Out)
```
Message 1 → Queue → Message 1 (processed first)
Message 2 → Queue → Message 2 (processed second)
Message 3 → Queue → Message 3 (processed third)
```

### Partition-based Ordering
```python
# Messages with same key go to same partition
def get_partition(message_key, num_partitions):
    return hash(message_key) % num_partitions

# Ensures ordering within partition
user_messages = produce_to_partition(user_id, message)
```

### Global Ordering
- Single partition/queue for strict ordering
- **Trade-off**: Limits parallelism and throughput

## Popular Message Queue Technologies

### Apache Kafka
- **Type**: Distributed streaming platform
- **Strengths**: High throughput, durability, scalability
- **Use Cases**: Event streaming, log aggregation, real-time analytics

#### Kafka Architecture
```
Producer → Broker 1 (Partition 0, 1)
        → Broker 2 (Partition 2, 3) → Consumer Group
        → Broker 3 (Partition 4, 5)
```

#### Kafka Concepts
```python
# Topic: Category of messages
topic = "user-events"

# Partition: Ordered sequence within topic
partition_0 = [msg1, msg2, msg3]
partition_1 = [msg4, msg5, msg6]

# Consumer Group: Load balancing consumers
consumer_group = "analytics-service"
```

### RabbitMQ
- **Type**: Traditional message broker
- **Strengths**: Flexible routing, reliability, management UI
- **Use Cases**: Task queues, RPC, complex routing

#### RabbitMQ Components
```
Producer → Exchange → Queue → Consumer
```

#### Exchange Types
```python
# Direct Exchange: Route by routing key
exchange_type = "direct"
routing_key = "user.created"

# Topic Exchange: Pattern matching
routing_pattern = "user.*"  # Matches user.created, user.updated

# Fanout Exchange: Broadcast to all queues
exchange_type = "fanout"  # No routing key needed
```

### Amazon SQS
- **Type**: Managed message queue service
- **Strengths**: Fully managed, scalable, integrated with AWS
- **Types**: Standard (at-least-once) and FIFO (exactly-once)

#### SQS Features
```python
# Standard Queue
queue_type = "standard"
delivery_guarantee = "at-least-once"
throughput = "unlimited"

# FIFO Queue
queue_type = "fifo"
delivery_guarantee = "exactly-once"
throughput = "3000 messages/second"
```

### Redis Pub/Sub
- **Type**: In-memory pub/sub system
- **Strengths**: Low latency, simple setup
- **Limitations**: No persistence, no delivery guarantees

```python
# Publisher
redis_client.publish("notifications", "New message")

# Subscriber
pubsub = redis_client.pubsub()
pubsub.subscribe("notifications")
for message in pubsub.listen():
    process_notification(message)
```

## Message Queue Design Patterns

### Work Queue Pattern
```python
# Producer adds tasks to queue
def add_task(task_data):
    queue.put({
        'id': generate_id(),
        'data': task_data,
        'timestamp': time.now()
    })

# Multiple workers process tasks
def worker():
    while True:
        task = queue.get()
        process_task(task)
        queue.task_done()
```

### Dead Letter Queue
```python
class MessageProcessor:
    def __init__(self):
        self.max_retries = 3
        self.dead_letter_queue = DeadLetterQueue()
    
    def process_message(self, message):
        try:
            # Process message
            business_logic(message)
        except Exception as e:
            if message.retry_count < self.max_retries:
                message.retry_count += 1
                queue.put(message)  # Retry
            else:
                self.dead_letter_queue.put(message)  # Give up
```

### Circuit Breaker Pattern
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.state = "CLOSED"  # CLOSED, OPEN, HALF_OPEN
    
    def call_service(self, message):
        if self.state == "OPEN":
            if time.now() - self.last_failure > self.timeout:
                self.state = "HALF_OPEN"
            else:
                raise Exception("Circuit breaker is OPEN")
        
        try:
            result = external_service.process(message)
            if self.state == "HALF_OPEN":
                self.state = "CLOSED"
                self.failure_count = 0
            return result
        except Exception as e:
            self.failure_count += 1
            if self.failure_count >= self.failure_threshold:
                self.state = "OPEN"
                self.last_failure = time.now()
            raise e
```

## Message Serialization

### JSON
```json
{
  "eventType": "user.created",
  "userId": 12345,
  "timestamp": "2024-01-01T12:00:00Z",
  "data": {
    "email": "user@example.com",
    "name": "John Doe"
  }
}
```
- **Pros**: Human readable, widely supported
- **Cons**: Larger size, slower parsing

### Protocol Buffers (protobuf)
```protobuf
message UserEvent {
  string event_type = 1;
  int64 user_id = 2;
  int64 timestamp = 3;
  UserData data = 4;
}

message UserData {
  string email = 1;
  string name = 2;
}
```
- **Pros**: Compact, fast, schema evolution
- **Cons**: Not human readable, requires schema

### Apache Avro
```json
{
  "type": "record",
  "name": "UserEvent",
  "fields": [
    {"name": "eventType", "type": "string"},
    {"name": "userId", "type": "long"},
    {"name": "timestamp", "type": "long"}
  ]
}
```
- **Pros**: Schema evolution, compact
- **Cons**: Requires schema registry

## Scaling Message Queues

### Horizontal Scaling

#### Kafka Partitioning
```python
# Scale by adding more partitions
topic_config = {
    "name": "user-events",
    "partitions": 10,  # Increased from 3
    "replication_factor": 3
}

# Scale consumers by adding to consumer group
consumer_group = "analytics-service"
consumers = [
    Consumer(group=consumer_group, partition=0),
    Consumer(group=consumer_group, partition=1),
    Consumer(group=consumer_group, partition=2)
]
```

#### RabbitMQ Clustering
```
Node 1: Queue A (master), Queue B (replica)
Node 2: Queue B (master), Queue C (replica)
Node 3: Queue C (master), Queue A (replica)
```

### Vertical Scaling
- Increase broker resources (CPU, memory, disk)
- Optimize configuration parameters
- Use faster storage (SSD vs HDD)

## Performance Optimization

### Batching
```python
# Producer batching
batch = []
for message in messages:
    batch.append(message)
    if len(batch) >= batch_size:
        producer.send_batch(batch)
        batch = []

# Consumer batching
messages = consumer.poll(max_records=100)
process_batch(messages)
```

### Compression
```python
# Kafka compression
producer_config = {
    'compression.type': 'gzip',  # or 'snappy', 'lz4'
    'batch.size': 16384,
    'linger.ms': 5
}
```

### Async Processing
```python
import asyncio

async def async_producer():
    tasks = []
    for message in messages:
        task = asyncio.create_task(send_message(message))
        tasks.append(task)
    
    await asyncio.gather(*tasks)
```

## Monitoring and Observability

### Key Metrics

#### Producer Metrics
- **Throughput**: Messages/second produced
- **Latency**: Time to send message
- **Error Rate**: Failed sends percentage
- **Queue Depth**: Messages waiting to be sent

#### Consumer Metrics
- **Lag**: Messages behind current offset
- **Throughput**: Messages/second consumed
- **Processing Time**: Time to process message
- **Error Rate**: Failed processing percentage

#### Broker Metrics
- **Disk Usage**: Storage utilization
- **Memory Usage**: Buffer utilization
- **Network I/O**: Bytes in/out per second
- **Partition Distribution**: Load balancing

### Monitoring Tools
```python
# Prometheus metrics
from prometheus_client import Counter, Histogram, Gauge

messages_produced = Counter('messages_produced_total')
message_processing_time = Histogram('message_processing_seconds')
queue_depth = Gauge('queue_depth_current')

def process_message(message):
    start_time = time.time()
    try:
        # Process message
        business_logic(message)
        messages_produced.inc()
    finally:
        processing_time = time.time() - start_time
        message_processing_time.observe(processing_time)
```

## Best Practices

### Message Design
- **Idempotent**: Safe to process multiple times
- **Self-contained**: Include all necessary data
- **Versioned**: Support schema evolution
- **Timestamped**: Include creation time

### Error Handling
- **Retry Logic**: Exponential backoff
- **Dead Letter Queues**: Handle poison messages
- **Circuit Breakers**: Prevent cascade failures
- **Monitoring**: Track error rates and patterns

### Security
- **Authentication**: Verify producer/consumer identity
- **Authorization**: Control access to topics/queues
- **Encryption**: Encrypt messages in transit and at rest
- **Network Security**: Use VPCs, security groups

## Interview Questions

1. **Design Choice**: When would you use Kafka vs RabbitMQ vs SQS?
2. **Ordering**: How do you ensure message ordering in a distributed system?
3. **Reliability**: How do you handle message loss and duplication?
4. **Scaling**: How do you scale message processing to handle 10x traffic?
5. **Monitoring**: What metrics would you track for a message queue system?

## Real-World Use Cases

### Netflix
- **Kafka**: Event streaming, real-time analytics
- **Use Case**: User activity tracking, recommendation updates
- **Scale**: Trillions of events per day

### Uber
- **Kafka**: Trip events, location updates
- **RabbitMQ**: Task queues, notifications
- **Use Case**: Real-time pricing, driver matching

### LinkedIn
- **Kafka**: Activity feeds, messaging
- **Use Case**: News feed updates, connection notifications
- **Scale**: Billions of messages per day

### Airbnb
- **RabbitMQ**: Booking workflows
- **Kafka**: Analytics events
- **Use Case**: Payment processing, search indexing