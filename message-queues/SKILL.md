---
name: message-queues
description: "Implement message queues, event-driven architecture, and asynchronous processing with RabbitMQ, Kafka, Redis Streams, and other message brokers."
---

# Message Queues Skill

Build scalable event-driven systems with message queues and asynchronous processing.

## When to Use

Use this skill when the user wants to:
- Implement message queues for asynchronous processing
- Build event-driven architectures
- Decouple services with message brokers
- Handle background jobs and task queues
- Implement pub/sub patterns
- Process streams of data
- Build reliable distributed systems
- Handle high-throughput data processing
- Implement CQRS and event sourcing patterns

## Message Queue Technologies

### RabbitMQ
- **Protocol**: AMQP (Advanced Message Queuing Protocol)
- **Use Cases**: Task queues, pub/sub, RPC, routing
- **Features**: Message acknowledgment, dead letter queues, priorities
- **Patterns**: Work queues, publish/subscribe, routing, topics, RPC

### Apache Kafka
- **Protocol**: Custom binary protocol over TCP
- **Use Cases**: Event streaming, log aggregation, real-time analytics
- **Features**: High throughput, distributed, fault-tolerant, replay capability
- **Patterns**: Event sourcing, CQRS, stream processing

### Redis Streams
- **Protocol**: Redis protocol (RESP)
- **Use Cases**: Lightweight messaging, real-time data streams
- **Features**: Consumer groups, message acknowledgment, persistence
- **Patterns**: Pub/sub, streams, lists as queues

### AWS SQS/SNS
- **Type**: Managed cloud service
- **Use Cases**: Serverless architectures, cloud-native apps
- **Features**: Fully managed, scalable, integrated with AWS services
- **Patterns**: Queue-based load leveling, fan-out

### Other Options
- **NATS**: High-performance messaging
- **Apache Pulsar**: Unified messaging and streaming
- **ActiveMQ**: Java-based message broker
- **ZeroMQ**: Lightweight messaging library

## Common Patterns

### 1. Work Queue (Task Queue)
Distribute tasks among workers for parallel processing.

```python
# Producer (RabbitMQ)
import pika

connection = pika.BlockingConnection(pika.ConnectionParameters('localhost'))
channel = connection.channel()
channel.queue_declare(queue='task_queue', durable=True)

channel.basic_publish(
    exchange='',
    routing_key='task_queue',
    body='Task data',
    properties=pika.BasicProperties(
        delivery_mode=2,  # Make message persistent
    )
)
```

```python
# Consumer (Worker)
def callback(ch, method, properties, body):
    print(f"Processing task: {body}")
    # Process task
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_qos(prefetch_count=1)
channel.basic_consume(queue='task_queue', on_message_callback=callback)
channel.start_consuming()
```

### 2. Publish/Subscribe
Broadcast messages to multiple consumers.

```python
# Publisher
channel.exchange_declare(exchange='logs', exchange_type='fanout')
channel.basic_publish(exchange='logs', routing_key='', body='Log message')

# Subscriber
result = channel.queue_declare(queue='', exclusive=True)
queue_name = result.method.queue
channel.queue_bind(exchange='logs', queue=queue_name)
```

### 3. Topic-Based Routing
Route messages based on patterns.

```python
# Publisher with routing key
channel.exchange_declare(exchange='topics', exchange_type='topic')
channel.basic_publish(
    exchange='topics',
    routing_key='user.created',
    body='User data'
)

# Consumer with pattern
channel.queue_bind(exchange='topics', queue=queue_name, routing_key='user.*')
```

### 4. Event Sourcing with Kafka
Store all changes as events.

```python
from kafka import KafkaProducer, KafkaConsumer
import json

# Producer
producer = KafkaProducer(
    bootstrap_servers=['localhost:9092'],
    value_serializer=lambda v: json.dumps(v).encode('utf-8')
)

event = {
    'event_type': 'OrderCreated',
    'order_id': '12345',
    'timestamp': '2024-01-01T00:00:00Z',
    'data': {'customer_id': '67890', 'total': 99.99}
}

producer.send('order-events', value=event)

# Consumer
consumer = KafkaConsumer(
    'order-events',
    bootstrap_servers=['localhost:9092'],
    value_deserializer=lambda m: json.loads(m.decode('utf-8')),
    auto_offset_reset='earliest',
    group_id='order-processor'
)

for message in consumer:
    event = message.value
    # Process event
    print(f"Processing {event['event_type']}: {event['order_id']}")
```

### 5. Dead Letter Queue
Handle failed messages.

```python
# Declare DLQ
channel.exchange_declare(exchange='dlx', exchange_type='fanout')
channel.queue_declare(queue='dead_letter_queue')
channel.queue_bind(exchange='dlx', queue='dead_letter_queue')

# Main queue with DLQ
args = {
    'x-dead-letter-exchange': 'dlx',
    'x-message-ttl': 60000  # 60 seconds
}
channel.queue_declare(queue='main_queue', arguments=args)
```

## Best Practices

### Message Design
- **Idempotent consumers**: Handle duplicate messages gracefully
- **Small messages**: Keep message payloads small, reference large data
- **Schema versioning**: Version your message schemas
- **Include metadata**: Timestamps, correlation IDs, message IDs

### Reliability
- **Acknowledgments**: Use manual acknowledgment for critical messages
- **Persistence**: Enable message persistence for durability
- **Retries**: Implement exponential backoff for retries
- **Dead letter queues**: Handle poison messages
- **Monitoring**: Monitor queue depth, consumer lag, throughput

### Performance
- **Batching**: Batch messages when possible
- **Prefetch**: Configure consumer prefetch count
- **Partitioning**: Use partitions for parallel processing (Kafka)
- **Connection pooling**: Reuse connections
- **Async processing**: Use async consumers when appropriate

### Security
- **Authentication**: Use credentials, SASL, or certificates
- **Encryption**: Enable TLS for data in transit
- **Authorization**: Control access to queues and topics
- **Network isolation**: Use VPCs or private networks

## Architecture Patterns

### Microservices Communication
```
Service A → Queue → Service B
         ↘ Queue → Service C
```

### Event-Driven Architecture
```
Event Producer → Event Bus → Event Consumers
                           ↘ Event Store
```

### CQRS (Command Query Responsibility Segregation)
```
Command → Command Bus → Write Model → Events
Events → Event Bus → Read Model (Projections)
```

### Saga Pattern
Distributed transactions across services using events.

```python
# Order Saga Orchestrator
def order_saga(order_data):
    # Step 1: Reserve inventory
    publish_event('inventory.reserve', order_data)

    # Step 2: Process payment
    publish_event('payment.process', order_data)

    # Step 3: Ship order
    publish_event('shipping.create', order_data)

# Compensation on failure
def compensate_order(order_id, failed_step):
    if failed_step >= 2:
        publish_event('payment.refund', {'order_id': order_id})
    if failed_step >= 1:
        publish_event('inventory.release', {'order_id': order_id})
```

## Error Handling

### Retry Strategies
```python
import time
from functools import wraps

def retry_with_backoff(max_retries=3, base_delay=1):
    def decorator(func):
        @wraps(func)
        def wrapper(*args, **kwargs):
            for attempt in range(max_retries):
                try:
                    return func(*args, **kwargs)
                except Exception as e:
                    if attempt == max_retries - 1:
                        raise
                    delay = base_delay * (2 ** attempt)
                    print(f"Retry {attempt + 1}/{max_retries} after {delay}s")
                    time.sleep(delay)
        return wrapper
    return decorator

@retry_with_backoff(max_retries=3)
def process_message(message):
    # Process message
    pass
```

### Circuit Breaker
```python
class CircuitBreaker:
    def __init__(self, failure_threshold=5, timeout=60):
        self.failure_count = 0
        self.failure_threshold = failure_threshold
        self.timeout = timeout
        self.last_failure_time = None
        self.state = 'CLOSED'  # CLOSED, OPEN, HALF_OPEN

    def call(self, func, *args, **kwargs):
        if self.state == 'OPEN':
            if time.time() - self.last_failure_time > self.timeout:
                self.state = 'HALF_OPEN'
            else:
                raise Exception("Circuit breaker is OPEN")

        try:
            result = func(*args, **kwargs)
            self.on_success()
            return result
        except Exception as e:
            self.on_failure()
            raise

    def on_success(self):
        self.failure_count = 0
        self.state = 'CLOSED'

    def on_failure(self):
        self.failure_count += 1
        self.last_failure_time = time.time()
        if self.failure_count >= self.failure_threshold:
            self.state = 'OPEN'
```

## Monitoring & Observability

### Key Metrics
- **Queue depth**: Number of messages in queue
- **Consumer lag**: Time/count behind latest message
- **Throughput**: Messages per second
- **Error rate**: Failed message percentage
- **Processing time**: Message processing duration
- **DLQ size**: Dead letter queue depth

### Health Checks
```python
def check_queue_health():
    metrics = {
        'queue_depth': get_queue_depth('task_queue'),
        'consumer_count': get_consumer_count('task_queue'),
        'message_rate': get_message_rate('task_queue'),
    }

    if metrics['queue_depth'] > 10000:
        alert("High queue depth", metrics)

    if metrics['consumer_count'] == 0:
        alert("No active consumers", metrics)

    return metrics
```

## Quick Start Examples

### RabbitMQ with Python (Pika)
```bash
# Install
pip install pika

# Run RabbitMQ with Docker
docker run -d --name rabbitmq -p 5672:5672 -p 15672:15672 rabbitmq:management
```

### Kafka with Python (kafka-python)
```bash
# Install
pip install kafka-python

# Run Kafka with Docker
docker-compose up -d  # Using kafka docker-compose.yml
```

### Redis Streams with Python (redis-py)
```bash
# Install
pip install redis

# Run Redis with Docker
docker run -d --name redis -p 6379:6379 redis
```

## Testing

### Unit Testing
```python
import pytest
from unittest.mock import Mock, patch

def test_message_processing():
    mock_channel = Mock()
    message = b'{"task": "process_data"}'

    process_message(mock_channel, None, None, message)

    mock_channel.basic_ack.assert_called_once()
```

### Integration Testing
```python
@pytest.fixture
def rabbitmq_container():
    # Use testcontainers or docker-compose for testing
    container = start_rabbitmq_container()
    yield container
    container.stop()

def test_message_flow(rabbitmq_container):
    # Publish message
    publish('test_queue', 'test message')

    # Consume message
    message = consume('test_queue', timeout=5)

    assert message == 'test message'
```

## Common Use Cases

1. **Background Job Processing**: Email sending, image processing, report generation
2. **Microservices Communication**: Async service-to-service messaging
3. **Event-Driven Systems**: React to business events across services
4. **Log Aggregation**: Collect and process logs from multiple sources
5. **Real-Time Analytics**: Stream processing and aggregation
6. **Order Processing**: E-commerce order workflows
7. **Notification Systems**: Push notifications, SMS, email queues
8. **Data Pipeline**: ETL processes, data transformation

## Deliverables

- Message queue setup and configuration
- Producer and consumer implementations
- Error handling and retry logic
- Dead letter queue handling
- Monitoring and alerting setup
- Documentation and runbooks
- Testing suite

## Quality Checklist

- Messages are processed idempotently
- Dead letter queues are configured
- Monitoring and alerting is in place
- Error handling includes retries
- Consumers acknowledge messages properly
- Message persistence is configured
- Security (TLS, auth) is enabled
- Documentation includes failure scenarios
- Load testing has been performed
- Graceful shutdown is implemented
