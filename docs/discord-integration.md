# Discord Webhook Integration

AlertManager doesn't natively support Discord, but you can easily integrate Discord notifications using webhooks.

## Method 1: Direct Discord Webhook (Simple)

### Step 1: Get Discord Webhook URL
1. In your Discord server, go to Server Settings → Integrations → Webhooks
2. Create New Webhook
3. Copy the webhook URL (looks like: `https://discord.com/api/webhooks/123456789/abcdef...`)

### Step 2: Update AlertManager Configuration

Replace the webhook URLs in `alertmanager/alertmanager.yml`:

```yaml
receivers:
  - name: 'critical-webhook'
    webhook_configs:
      - url: 'https://discord.com/api/webhooks/YOUR_WEBHOOK_ID/YOUR_TOKEN'
        send_resolved: true
        http_config:
          headers:
            Content-Type: application/json
        # Discord webhook format
        title: 'Homelab Alert'
        text: |
          {
            "content": "🚨 **{{ .Status | toUpper }}**: {{ .GroupLabels.alertname }}\n**Instance**: {{ .CommonLabels.instance }}\n**Description**: {{ range .Alerts }}{{ .Annotations.description }}{{ end }}"
          }
```

## Method 2: Webhook Bridge (Recommended)

For better formatting and more control, use a webhook bridge service.

### Using webhook.site for testing:
1. Go to https://webhook.site
2. Copy your unique URL
3. Update AlertManager config with that URL
4. You'll see all webhook calls in the webhook.site interface

### Using a Discord webhook bridge:
Deploy a simple service that transforms AlertManager webhooks into Discord-formatted messages:

```yaml
# Add to docker-compose.yml
  webhook-bridge:
    image: alertmanager-discord:latest  # Custom image or webhook bridge
    container_name: webhook-bridge
    ports:
      - "5001:5001"
    environment:
      - DISCORD_WEBHOOK_URL=https://discord.com/api/webhooks/YOUR_ID/YOUR_TOKEN
    networks:
      - monitoring
```

## Method 3: Using ntfy.sh (Alternative)

For a simpler notification system:

1. Install ntfy.sh app on your phone
2. Subscribe to a topic like "homelab-alerts"
3. Update AlertManager:

```yaml
receivers:
  - name: 'ntfy-alerts'
    webhook_configs:
      - url: 'https://ntfy.sh/homelab-alerts'
        send_resolved: true
        http_config:
          headers:
            Title: "Homelab Alert"
            Priority: "high"
            Tags: "warning,computer"
```

## Testing Alerts

Once configured, test your setup:

```bash
# Force an alert by stopping a service
docker stop node-exporter

# Wait 2-3 minutes for alert to fire
# Check AlertManager UI: http://localhost:9093

# Restart the service
docker start node-exporter
```

## Troubleshooting

- **Webhook not firing**: Check AlertManager logs with `docker-compose logs alertmanager`
- **Discord not receiving**: Verify webhook URL and test with curl
- **Message format issues**: Discord expects JSON with specific format

## Current Configuration

Your current setup uses `https://httpbin.org/post` as a test endpoint. You can see webhook calls at:
- AlertManager UI: http://localhost:9093
- Check what's being sent in the AlertManager logs