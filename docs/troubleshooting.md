# Troubleshooting Guide

## Common Issues and Solutions

### Docker Compose Issues

#### Services Won't Start
```bash
# Check logs
docker-compose logs [service-name]

# Common issues:
# 1. Port conflicts
netstat -tlnp | grep :3000  # Check if port is in use

# 2. Permission issues
sudo chown -R 472:472 grafana_data/  # Fix Grafana permissions

# 3. Configuration errors
docker-compose config  # Validate compose file
```

#### Out of Disk Space
```bash
# Clean up Docker
docker system prune -a

# Remove old volumes
docker volume prune
```

### Prometheus Issues

#### Targets Not Discovered
```bash
# Check Prometheus targets page: http://localhost:9090/targets

# Common fixes:
# 1. Check IP addresses in prometheus.yml
# 2. Verify services are running on target machines
# 3. Check firewall rules
sudo ufw allow 9100  # Allow Node Exporter port
```

#### Home Assistant Integration Not Working
```bash
# Verify Home Assistant prometheus integration
# Check Home Assistant logs for authentication errors

# Update bearer token in prometheus.yml
# Generate new long-lived access token if needed
```

#### Blackbox Exporter Failures
```bash
# Check if websites are accessible
curl -I https://your-website.com

# Check blackbox configuration
docker-compose logs blackbox-exporter

# Verify DNS resolution
nslookup your-website.com
```

### Grafana Issues

#### Dashboard Not Loading Data
```bash
# Check data source connection
# Grafana → Configuration → Data Sources → Test

# Common fixes:
# 1. Verify Prometheus URL (should be http://prometheus:9090)
# 2. Check if Prometheus is collecting data
# 3. Verify metric names in queries
```

#### Dashboards Not Auto-Loading
```bash
# Check provisioning logs
docker-compose logs grafana

# Verify file permissions
ls -la grafana/dashboards/
ls -la grafana/provisioning/

# Restart Grafana
docker-compose restart grafana
```

### Node Exporter Issues

#### Metrics Not Available
```bash
# Check if Node Exporter is running
curl http://target-ip:9100/metrics

# On target machine:
sudo systemctl status node_exporter
sudo systemctl restart node_exporter

# Check logs
sudo journalctl -u node_exporter -f
```

#### Permission Denied Errors
```bash
# Common on systemd systems
# Ensure Node Exporter has correct permissions

# Check service file:
sudo systemctl cat node_exporter

# Verify user exists:
id node_exporter
```

### AlertManager Issues

#### Alerts Not Firing
```bash
# Check AlertManager status: http://localhost:9093

# Verify alert rules are loaded in Prometheus
# Go to http://localhost:9090/rules

# Check if metrics match alert conditions
# Use Prometheus query browser to test expressions
```

#### Notifications Not Sent
```bash
# Check AlertManager logs
docker-compose logs alertmanager

# Test webhook manually:
curl -X POST YOUR_DISCORD_WEBHOOK_URL \
  -H "Content-Type: application/json" \
  -d '{"content": "Test message"}'

# Verify routing configuration in alertmanager.yml
```

### Network Issues

#### Services Can't Communicate
```bash
# Check Docker network
docker network ls
docker network inspect home-lab-management_monitoring

# Test connectivity between containers
docker exec prometheus ping grafana
docker exec grafana ping prometheus
```

#### External Services Unreachable
```bash
# Test from host
ping target-ip
telnet target-ip 9100

# Test from container
docker exec prometheus ping target-ip
docker exec prometheus telnet target-ip 9100
```

### Performance Issues

#### High Memory Usage
```bash
# Check container resource usage
docker stats

# Adjust retention periods in prometheus.yml:
# --storage.tsdb.retention.time=15d
# --storage.tsdb.retention.size=10GB
```

#### Slow Dashboard Loading
```bash
# Reduce query intervals in dashboards
# Use recording rules for complex queries
# Increase Grafana memory limit in docker-compose.yml
```

### Data Issues

#### Missing Historical Data
```bash
# Check Prometheus data retention
# Data older than retention period is automatically deleted

# Check storage usage
du -sh prometheus_data/
```

#### Incorrect Metrics
```bash
# Verify time synchronization across all systems
timedatectl status

# Check for clock skew
# Prometheus requires accurate time across all monitored systems
```

## Getting Help

### Log Analysis
```bash
# View all logs
docker-compose logs

# Follow specific service logs
docker-compose logs -f prometheus

# Search logs for errors
docker-compose logs | grep -i error
```

### Health Checks
```bash
# Quick health check script
#!/bin/bash
echo "=== Docker Compose Status ==="
docker-compose ps

echo "=== Prometheus Targets ==="
curl -s http://localhost:9090/api/v1/targets | jq '.data.activeTargets[] | {job: .labels.job, instance: .labels.instance, health: .health}'

echo "=== Grafana Health ==="
curl -s http://localhost:3000/api/health

echo "=== AlertManager Status ==="
curl -s http://localhost:9093/api/v1/status
```

### Reset Everything
```bash
# Nuclear option - reset all data
docker-compose down -v
docker-compose up -d

# This will lose all historical data and custom configurations
```

## Prevention

### Regular Maintenance
- Monitor disk space usage
- Update container images monthly
- Backup Grafana dashboards and configurations
- Test alert notifications regularly
- Review and tune alert thresholds

### Monitoring the Monitoring
- Set up external monitoring for the monitoring stack itself
- Use a separate simple uptime monitor
- Configure alerts for monitoring stack failures