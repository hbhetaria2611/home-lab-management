# Homelab Monitoring Stack

A comprehensive monitoring solution for distributed homelab setups using Grafana, Prometheus, and AlertManager. Perfect for monitoring Raspberry Pi devices, Home Assistant, external websites, and various homelab services.

> ⚠️ **Security Note**: Before deploying, copy `.env.example` to `.env` and configure with your actual values. Never commit real tokens or credentials to version control.

## Architecture Overview

### Monitored Services
- **Raspberry Pi Services**:
  - Home Assistant
  - Nginx Proxy Manager
  - Vaultwarden
- **External Services**:
  - Homebridge (separate device)
  - Cloudflare websites (portfolio + meal planning)
- **System Metrics**:
  - CPU, Memory, Disk usage
  - Network performance
  - Temperature monitoring

### Monitoring Stack
- **Prometheus**: Metrics collection and storage
- **Grafana**: Visualization and dashboards
- **AlertManager**: Alert routing and notifications
- **Node Exporter**: System metrics collection
- **Blackbox Exporter**: External service monitoring

## Quick Start

1. **Deploy the monitoring stack**:
   ```bash
   docker-compose up -d
   ```

2. **Access dashboards**:
   - Grafana: http://localhost:3000 (admin/admin)
   - Prometheus: http://localhost:9090
   - AlertManager: http://localhost:9093

3. **Configure targets**:
   - Update `prometheus/prometheus.yml` with your service endpoints
   - Import pre-built dashboards in Grafana

## Directory Structure

```
├── docker-compose.yml          # Main stack deployment
├── prometheus/
│   ├── prometheus.yml         # Prometheus configuration
│   └── alerts.yml            # Alert rules
├── grafana/
│   ├── dashboards/           # Pre-built dashboards
│   └── provisioning/         # Auto-provisioning config
├── alertmanager/
│   └── alertmanager.yml      # Alert routing configuration
└── docs/
    ├── setup-guide.md        # Detailed setup instructions
    └── troubleshooting.md    # Common issues and solutions
```

## Key Features

- **Multi-device monitoring** across RPi and external services
- **Pre-configured dashboards** for homelab services
- **Smart alerting** with Discord/Slack/email notifications
- **Security monitoring** for failed logins and certificate expiry
- **Performance tracking** for all critical services
- **Easy deployment** with Docker Compose

## Next Steps

1. Customize `prometheus/prometheus.yml` with your service IPs
2. Set up alert destinations in `alertmanager/alertmanager.yml`
3. Configure Home Assistant integration for entity monitoring
4. Set up Cloudflare API monitoring for website performance