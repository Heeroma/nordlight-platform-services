# Logging placeholder - configure Loki and Promtail here

## Loki (Log Aggregation)
- Uncomment loki service in docker-compose.yml
- Access: http://localhost:3100
- Query logs in Grafana

## Promtail (Log Shipping)
- Uncomment promtail service in docker-compose.yml
- Tails Docker container logs
- Ships to Loki
