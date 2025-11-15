# RabbitMQ Message Queue Configuration

## Access

- AMQP: localhost:5672
- Management UI: http://localhost:15672
- Credentials: admin/fasttrader2024

## Use Cases

- **Event-driven communication** between microservices
- **Task queues** for background processing
- **Pub/Sub** for broadcasting events
- **Request/Reply** patterns

## Example: Publish Message

```python
import pika

connection = pika.BlockingConnection(
    pika.ConnectionParameters(
        host='localhost',
        port=5672,
        credentials=pika.PlainCredentials('admin', 'fasttrader2024')
    )
)
channel = connection.channel()

# Declare queue
channel.queue_declare(queue='etl_jobs', durable=True)

# Publish message
channel.basic_publish(
    exchange='',
    routing_key='etl_jobs',
    body='Run EOD ETL',
    properties=pika.BasicProperties(delivery_mode=2)  # Persistent
)

connection.close()
```

## Example: Consume Messages

```python
def callback(ch, method, properties, body):
    print(f"Received: {body}")
    # Process message
    ch.basic_ack(delivery_tag=method.delivery_tag)

channel.basic_consume(queue='etl_jobs', on_message_callback=callback)
channel.start_consuming()
```

## Common Exchanges

- **fanout** - Broadcast to all queues
- **direct** - Route by routing key
- **topic** - Route by pattern matching
- **headers** - Route by message headers

## Enable in docker-compose.yml

Uncomment the rabbitmq service section.
