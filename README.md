# FastTrader Platform Services

Central infrastructure and cross-cutting services for the FastTrader microservices ecosystem.

## Services Included

### 🔍 Monitoring Stack (Active)
- **Prometheus** (port 9090) - Metrics collection and alerting
- **Grafana** (port 3010) - Visualization and dashboards
- **Alertmanager** (port 9093) - Alert routing and notifications

### 🗄️ Shared Infrastructure (Active)
- **Redis** (port 6379) - Shared cache, pub/sub, sessions

### 🔒 Security Stack (Optional)
- **Vault** (port 8200) - Secrets management
- **Nginx** (ports 80/443) - API Gateway, rate limiting, SSL termination

### 📊 Logging Stack (Optional)
- **Loki** (port 3100) - Log aggregation
- **Promtail** - Log shipping from all services

### 🔍 Tracing Stack (Optional)
- **Jaeger** (port 16686) - Distributed tracing

### 🌐 Service Discovery (Optional)
- **Consul** (port 8500) - Service registry, health checks, KV store

### 📬 Messaging (Optional)
- **RabbitMQ** (ports 5672/15672) - Message queue with management UI

### 📈 System Metrics (Optional)
- **Node Exporter** (port 9100) - Host system metrics

## Quick Start

### Start Core Services (Monitoring Only)
```powershell
cd c:/Users/heero/Python/fasttrader/services/platform-services
docker-compose up -d
```

This starts:
- Prometheus: http://localhost:9090
- Grafana: http://localhost:3010 (admin/fasttrader2024)
- Alertmanager: http://localhost:9093
- Redis: localhost:6379

### Enable Optional Services

Edit `docker-compose.yml` and uncomment the services you need:
- **Logging** (Loki + Promtail) - Centralized log aggregation
- **Tracing** (Jaeger) - Distributed request tracing
- **Security** (Vault + Nginx) - Secrets + API gateway
- **Service Discovery** (Consul) - Service registry
- **Messaging** (RabbitMQ) - Message queue
- **System Metrics** (Node Exporter) - Host metrics

### Stop All Services
```powershell
docker-compose down
```

### Stop and Remove All Data
```powershell
docker-compose down -v
```

## Service Configuration

### Monitored Services

Prometheus automatically scrapes metrics from:
- **Data Platform** - http://host.docker.internal:8000/metrics
- **ML Platform** - http://host.docker.internal:8010/metrics
- **Regime Service** - http://host.docker.internal:8020/metrics
- **Streaming Service** - http://host.docker.internal:8030/metrics
- **Admin Portal** - http://host.docker.internal:8100/metrics

### Grafana Dashboards

Pre-configured dashboards:
- **Data Platform Dashboard** - Storage, ETL, Quality metrics
- More dashboards coming for ML, Regime, Streaming services

Access: http://localhost:3010 → Dashboards → FastTrader folder

### Alerting

Alerts are configured per service in `monitoring/prometheus/alerts/`:
- `data-platform.yml` - Data Platform alerts
- Add more service-specific alert files here

Alert routing configured in `monitoring/alertmanager/config.yml`

## Adding a New Service

### 1. Add /metrics Endpoint to Service

```python
# In your service's main.py
from prometheus_client import generate_latest, CONTENT_TYPE_LATEST
from fastapi import FastAPI
from fastapi.responses import PlainTextResponse

app = FastAPI()

@app.get("/metrics")
async def metrics():
    metrics_data = generate_latest()
    return PlainTextResponse(
        content=metrics_data.decode('utf-8'),
        media_type=CONTENT_TYPE_LATEST
    )
```

### 2. Add Scrape Config to Prometheus

Edit `monitoring/prometheus/prometheus.yml`:

```yaml
scrape_configs:
  - job_name: 'your-service'
    static_configs:
      - targets: ['host.docker.internal:YOUR_PORT']
    metrics_path: '/metrics'
    scrape_interval: 15s
```

### 3. Create Alert Rules

Create `monitoring/prometheus/alerts/your-service.yml`:

```yaml
groups:
  - name: your_service_alerts
    interval: 30s
    rules:
      - alert: YourServiceDown
        expr: up{job="your-service"} == 0
        for: 1m
        labels:
          severity: critical
          service: your-service
        annotations:
          summary: "Your service is down"
          description: "Your service has been down for more than 1 minute"
```

### 4. Create Grafana Dashboard

1. Create dashboard in Grafana UI
2. Export JSON via Share → Export
3. Save to `monitoring/grafana/dashboards/your-service.json`
4. Restart Grafana to auto-import

## Architecture

```
FastTrader Microservices
    │
    ├─► Data Platform :8000 ────┐
    ├─► ML Platform :8010 ───────┤
    ├─► Regime Service :8020 ────┤
    ├─► Streaming :8030 ─────────┤ /metrics endpoints
    └─► Admin Portal :8100 ──────┘
                                  │
                                  ▼
                            Prometheus :9090
                              (15s scrape)
                                  │
                    ┌─────────────┼─────────────┐
                    ▼             ▼             ▼
              Grafana :3010  Alertmanager  Alert Rules
              (dashboards)   (notifications) (evaluation)
```

## Metrics Collected

### Application Metrics
- API request rates
- Response times (p50, p95, p99)
- Error rates
- Business metrics (records processed, jobs run, etc.)

### Infrastructure Metrics
- Service health (up/down)
- Database connections
- Cache hit rates
- Storage usage

### Quality Metrics
- Data completeness
- Data freshness
- ETL success rates

## Alert Severity Levels

**Critical** 🚨
- Immediate notification
- Service down, data loss, security breach
- Repeat every 5 minutes until resolved

**Warning** ⚠️
- Delayed notification (30s)
- Performance degradation, approaching limits
- Repeat every 30 minutes

**Info** ℹ️
- Daily digest
- General status, low-priority notifications
- Repeat every 24 hours

## Accessing Services

| Service | URL | Credentials |
|---------|-----|-------------|
| Prometheus | http://localhost:9090 | None |
| Grafana | http://localhost:3010 | admin/fasttrader2024 |
| Alertmanager | http://localhost:9093 | None |
| Redis | localhost:6379 | None |
| Consul UI | http://localhost:8500 | None (when enabled) |
| Jaeger UI | http://localhost:16686 | None (when enabled) |
| RabbitMQ UI | http://localhost:15672 | admin/fasttrader2024 (when enabled) |
| Vault | http://localhost:8200 | Root token (when enabled) |

## Troubleshooting

### Prometheus Not Scraping
1. Check if target service is running and has /metrics endpoint
2. Verify target URL in `prometheus.yml`
3. Check Prometheus targets: http://localhost:9090/targets
4. Check Prometheus logs: `docker logs fasttrader-prometheus`

### Grafana Dashboard Empty
1. Verify Prometheus datasource: Configuration → Data Sources
2. Test connection should succeed
3. Check if metrics exist in Prometheus: http://localhost:9090/graph

### Alerts Not Firing
1. Check alert rules: http://localhost:9090/alerts
2. Verify Alertmanager is running: http://localhost:9093
3. Check Prometheus config points to Alertmanager
4. Review alert expression in Prometheus UI

## Production Deployment

### Security Hardening
1. Change Grafana admin password
2. Enable HTTPS for all services
3. Configure authentication for Prometheus
4. Set up Vault for secrets management
5. Enable nginx as API gateway with rate limiting

### Resource Limits
Add to `docker-compose.yml`:
```yaml
deploy:
  resources:
    limits:
      memory: 2G
      cpus: '1.0'
```

### Backup Strategy
- Prometheus: Snapshot `/prometheus` volume
- Grafana: Export dashboards to JSON, backup `/var/lib/grafana`
- Alertmanager: Backup `/alertmanager` volume

## Environment Variables

```bash
# Grafana
GF_SECURITY_ADMIN_PASSWORD=change-in-production
GF_SERVER_ROOT_URL=https://grafana.fasttrader.com

# Alertmanager (for notifications)
SLACK_WEBHOOK_URL=your-slack-webhook
SMTP_HOST=smtp.gmail.com
SMTP_USER=alerts@fasttrader.com
SMTP_PASSWORD=your-app-password
```

## Next Steps

1. ✅ Start monitoring stack
2. ✅ Verify all services are being scraped
3. ✅ Review Grafana dashboards
4. ⚙️ Configure alert notifications (Slack, email)
5. ⚙️ Enable logging stack (Loki + Promtail)
6. ⚙️ Enable tracing (Jaeger)
7. ⚙️ Set up Vault for secrets
8. ⚙️ Configure nginx as API gateway

## Links

- [Prometheus Documentation](https://prometheus.io/docs/)
- [Grafana Documentation](https://grafana.com/docs/)
- [Alertmanager Documentation](https://prometheus.io/docs/alerting/latest/alertmanager/)
- [Loki Documentation](https://grafana.com/docs/loki/latest/)
- [Jaeger Documentation](https://www.jaegertracing.io/docs/)
