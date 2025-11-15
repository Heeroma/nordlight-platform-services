# Consul Service Discovery Configuration

## Basic Setup

When enabled, Consul provides:
- Service registration and discovery
- Health checking
- Key/Value store for configuration
- UI for visualization

## Access

- UI: http://localhost:8500
- DNS: localhost:8600
- HTTP API: http://localhost:8500/v1/

## Register a Service

Services can register themselves with Consul:

```python
import consul

c = consul.Consul(host='localhost', port=8500)

# Register service
c.agent.service.register(
    name='data-platform',
    service_id='data-platform-1',
    address='host.docker.internal',
    port=8000,
    check=consul.Check.http('http://host.docker.internal:8000/health', interval='10s')
)
```

## Service Discovery

Other services can discover registered services:

```python
# Find data-platform instances
services = c.health.service('data-platform', passing=True)
for service in services:
    print(f"Service at {service['Service']['Address']}:{service['Service']['Port']}")
```

## Enable in docker-compose.yml

Uncomment the consul service section.
