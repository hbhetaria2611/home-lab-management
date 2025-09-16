# Homelab Monitoring Setup Guide

## Prerequisites

- Docker and Docker Compose installed
- Home Assistant with Prometheus integration enabled
- Node Exporter installed on devices you want to monitor
- Network access between monitoring stack and target devices

## Initial Setup

### 1. Clone and Configure

```bash
git clone <this-repo>
cd home-lab-management
```

### 2. Update Configuration Files

#### Update Prometheus targets in `prometheus/prometheus.yml`:

```yaml
# Replace these IPs with your actual device IPs
- targets: ['192.168.1.100:8123']  # Home Assistant
- targets: ['192.168.1.100:9100']  # RPi Node Exporter
- targets: ['192.168.1.101:9100']  # Homebridge device
```

#### Update website URLs in the blackbox monitoring section:
```yaml
- targets:
  - https://your-portfolio-site.com
  - https://your-meal-planning-site.com
```

### 3. Home Assistant Configuration

Enable Prometheus integration in Home Assistant by adding to `configuration.yaml`:

```yaml
prometheus:
  namespace: hass
  filter:
    include_domains:
      - sensor
      - binary_sensor
      - switch
      - light
      - climate
    include_entities:
      - sun.sun
```

Create a long-lived access token:
1. Go to your profile in Home Assistant
2. Scroll down to "Long-Lived Access Tokens"
3. Create a new token
4. Copy the token and update `prometheus/prometheus.yml`

### 4. Install Node Exporter on Target Devices

#### On Raspberry Pi (if not using Docker):
```bash
wget https://github.com/prometheus/node_exporter/releases/download/v1.3.1/node_exporter-1.3.1.linux-armv7.tar.gz
tar xvfz node_exporter-1.3.1.linux-armv7.tar.gz
sudo cp node_exporter-1.3.1.linux-armv7/node_exporter /usr/local/bin/
sudo useradd --no-create-home --shell /bin/false node_exporter
sudo chown node_exporter:node_exporter /usr/local/bin/node_exporter

# Create systemd service
sudo tee /etc/systemd/system/node_exporter.service > /dev/null <<EOF
[Unit]
Description=Node Exporter
Wants=network-online.target
After=network-online.target

[Service]
User=node_exporter
Group=node_exporter
Type=simple
ExecStart=/usr/local/bin/node_exporter

[Install]
WantedBy=multi-user.target
EOF

sudo systemctl daemon-reload
sudo systemctl enable node_exporter
sudo systemctl start node_exporter
```

#### On Homebridge device (macOS/Linux):
Similar process, or use Docker:
```bash
docker run -d --name node-exporter \
  --net="host" \
  --pid="host" \
  -v "/:/host:ro,rslave" \
  prom/node-exporter:latest \
  --path.rootfs=/host
```

### 5. Configure Nginx Proxy Manager Monitoring (Optional)

If you want to monitor Nginx Proxy Manager:

1. Add nginx-prometheus-exporter to your NPM setup
2. Update the target in prometheus.yml

### 6. Set Up Alerting

#### Discord Webhook:
1. Create a Discord server or use existing
2. Go to Server Settings → Integrations → Webhooks
3. Create New Webhook
4. Copy webhook URL
5. Update `alertmanager/alertmanager.yml` with your webhook URL

#### Email Alerts (Optional):
Uncomment and configure email sections in `alertmanager/alertmanager.yml`

## Deployment

### 1. Start the Stack

```bash
docker-compose up -d
```

### 2. Verify Services

Check that all services are running:
```bash
docker-compose ps
```

### 3. Access Dashboards

- **Grafana**: http://localhost:3000 (admin/admin)
- **Prometheus**: http://localhost:9090
- **AlertManager**: http://localhost:9093

### 4. Import Dashboards

1. Login to Grafana
2. The homelab overview dashboard should auto-load
3. Import additional dashboards from Grafana.com:
   - Node Exporter Full: 1860
   - Docker Container Metrics: 193
   - Home Assistant: 11693

### 5. Test Alerting

Trigger a test alert:
```bash
# Stop a service to test down alert
docker stop node-exporter

# Wait 2-3 minutes and check AlertManager
# Restart the service
docker start node-exporter
```

## Customization

### Adding New Services

1. Add new scrape config to `prometheus/prometheus.yml`
2. Add relevant alert rules to `prometheus/alerts.yml`
3. Restart Prometheus: `docker-compose restart prometheus`

### Creating Custom Dashboards

1. Create dashboard in Grafana UI
2. Export JSON
3. Save to `grafana/dashboards/`
4. Dashboard will auto-load on restart

### Modifying Alerts

1. Edit `prometheus/alerts.yml`
2. Reload Prometheus config: `curl -X POST http://localhost:9090/-/reload`

## Troubleshooting

See `docs/troubleshooting.md` for common issues and solutions.

## Security Considerations

- Change default Grafana password
- Use strong authentication for external access
- Consider running behind reverse proxy with SSL
- Regularly update container images
- Monitor access logs for suspicious activity