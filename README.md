## Up & Running

`docker-compose up -d`

### Kafka

Ìnside container (`docker-compose exec kafka bash`):

 * Create the topic
 
```bash
kafka-topics --create \
  --bootstrap-server kafka:9092 \
  --replication-factor 1 \
  --partitions 1 \
  --topic flight_delay_classification_request
```

