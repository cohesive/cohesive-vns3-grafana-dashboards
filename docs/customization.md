# Customization Guide

This guide helps you customize VNS3 Grafana dashboards for your specific monitoring needs and environment.

## 🎨 Visual Customization

### Color Schemes and Themes

#### Modify Panel Colors

1. **Edit Panel** → **Field** → **Standard Options** → **Color**
2. **Choose color mode:**
   - **Single color:** Fixed color for all series
   - **By series:** Different color per metric
   - **By value:** Color based on thresholds

#### Custom Color Palettes

```json
{
  "fieldConfig": {
    "defaults": {
      "color": {
        "mode": "palette-classic"
      }
    },
    "overrides": [
      {
        "matcher": {
          "id": "byName",
          "options": "cpu_user"
        },
        "properties": [
          {
            "id": "color",
            "value": {
              "mode": "fixed",
              "fixedColor": "#1f77b4"
            }
          }
        ]
      }
    ]
  }
}
```

### Threshold Configuration

#### System Metrics Thresholds

```json
{
  "thresholds": {
    "mode": "absolute",
    "steps": [
      {
        "color": "green",
        "value": null
      },
      {
        "color": "yellow",
        "value": 70
      },
      {
        "color": "red",
        "value": 90
      }
    ]
  }
}
```

#### Network Performance Thresholds

- **TCP Connections:** Yellow > 1000, Red > 5000
- **Packet Loss:** Yellow > 0.1%, Red > 1%
- **Retransmissions:** Yellow > 10/sec, Red > 100/sec

### Panel Layout and Sizing

#### Responsive Grid System

```json
{
  "gridPos": {
    "h": 8,    // Height in grid units
    "w": 12,   // Width in grid units
    "x": 0,    // X position
    "y": 0     // Y position
  }
}
```

#### Common Panel Sizes

- **Full width charts:** w=24, h=8-12
- **Half width panels:** w=12, h=6-8
- **Quarter panels (stats):** w=6, h=4
- **Sidebar info:** w=6, h=12+

## 🔧 Query Customization

### Adding Custom Metrics

#### SNMP OID Extensions

```toml
# In snmp.conf, add custom OIDs
[[inputs.snmp.field]]
  name = "custom_metric"
  oid = "1.3.6.1.4.1.2021.x.x.x"
```

#### VNS3 API Extensions

```toml
# In vns3-api.conf, add new endpoints
[[inputs.http]]
  urls = ["http://localhost:5000/vns3_custom"]
  data_format = "json_v2"
```

### Query Optimization

#### Efficient Time Ranges

```flux
// Good - Use dashboard time variables
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)

// Avoid - Fixed time ranges
from(bucket: "telegraf")
  |> range(start: -24h)
```

#### Aggregation Windows

```flux
// Optimize for dashboard refresh rate
|> aggregateWindow(every: v.windowPeriod, fn: mean, createEmpty: false)

// Custom aggregation for high-frequency data
|> aggregateWindow(every: 1m, fn: max, createEmpty: false)
```

### Multi-Controller Queries

#### Controller Filtering

```flux
// Use template variables for dynamic filtering
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["hostname"] == "${hostname}")
  |> filter(fn: (r) => r["topology"] == "${topology}")
```

#### Cross-Controller Comparisons

```flux
// Compare metrics across all controllers
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "snmp")
  |> filter(fn: (r) => r["_field"] == "cpu_idle")
  |> group(columns: ["hostname"])
  |> mean()
```

## 📊 Dashboard Templates

### Creating Template Variables

#### Hostname Variable

```json
{
  "name": "hostname",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "query": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"snmp\") |> keep(columns: [\"hostname\"]) |> distinct(column: \"hostname\")"
,
  "refresh": 1,
  "multi": false,
  "includeAll": false
}
```

#### Topology Variable with Dependencies

```json
{
  "name": "topology",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "query": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"snmp\") |> filter(fn: (r) => r.hostname == \"${hostname}\") |> keep(columns
: [\"customerPrefix\"]) |> distinct(column: \"customerPrefix\")",
  "refresh": 2
}
```

#### Multi-Select Variables

```json
{
  "name": "tunnels",
  "multi": true,
  "includeAll": true,
  "allValue": ".*",
  "query": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"vpn_tunnel\") |> keep(columns: [\"instance\"]) |> distinct(column: \"instan
ce\")"
}
```

### Custom Dashboard Rows

#### Collapsible Sections

```json
{
  "collapsed": true,
  "type": "row",
  "title": "Network Performance Details",
  "gridPos": {
    "h": 1,
    "w": 24,
    "x": 0,
    "y": 8
  }
}
```

## 🚨 Alerting Configuration

### Panel-Based Alerts

#### CPU Usage Alert

```json
{
  "alert": {
    "conditions": [
      {
        "evaluator": {
          "params": [80],
          "type": "gt"
        },
        "operator": {
          "type": "and"
        },
        "query": {
          "params": ["A", "5m", "now"]
        },
        "reducer": {
          "params": [],
          "type": "avg"
        },
        "type": "query"
      }
    ],
    "executionErrorState": "alerting",
    "for": "2m",
    "frequency": "10s",
    "handler": 1,
    "name": "High CPU Usage",
    "noDataState": "no_data"
  }
}
```

#### Tunnel Status Alert

```json
{
  "alert": {
    "conditions": [
      {
        "evaluator": {
          "params": [1],
          "type": "lt"
        },
        "query": {
          "queryType": "",
          "refId": "A"
        },
        "reducer": {
          "type": "last"
        },
        "type": "query"
      }
    ],
    "message": "IPsec tunnel ${tunnel} is down on ${hostname}",
    "name": "Tunnel Down Alert"
  }
}
```

### Notification Channels

#### Slack Integration

1. **Create** Slack webhook URL
2. **Add** notification channel in Grafana:
   - **Type:** Slack
   - **Webhook URL:** Your Slack webhook
   - **Channel:** #vns3-alerts
   - **Username:** VNS3-Monitor

#### Email Notifications

```json
{
  "type": "email",
  "settings": {
    "addresses": "ops-team@company.com;network-team@company.com",
    "subject": "VNS3 Alert: ${alertname}",
    "body": "Alert: ${message}\nController: ${hostname}\nTopology: ${topology}"
  }
}
```

## 🔗 Dashboard Links and Navigation

### Cross-Dashboard Navigation

#### Dashboard Links

```json
{
  "dashboardLinks": [
    {
      "title": "System Overview",
      "type": "dashboards",
      "tags": ["vns3", "system"],
      "targetBlank": false,
      "includeVars": true,
      "keepTime": true
    },
    {
      "title": "Network Performance",
      "type": "dashboard",
      "dashboard": "vns3-network-performance",
      "uri": "db/vns3-network-performance"
    }
  ]
}
```

#### Panel Links

```json
{
  "links": [
    {
      "title": "Drill Down to Tunnels",
      "url": "/d/vns3-ipsec-tunnels?var-hostname=${hostname}&var-topology=${topology}",
      "targetBlank": false
    }
  ]
}
```

### URL Parameter Passing

#### Auto-Population from Context

```javascript
// Pass current variable values to linked dashboard
/d/vns3-peers-links?var-hostname=${hostname}&var-topology=${topology}&from=${__from}&to=${__to}
```

## 📱 Mobile Optimization

### Responsive Panel Configuration

#### Mobile-Friendly Layouts

```json
{
  "panels": [
    {
      "gridPos": {
        "h": 6,
        "w": 24,  // Full width on mobile
        "x": 0,
        "y": 0
      },
      "options": {
        "legend": {
          "displayMode": "list",  // Compact legend
          "placement": "bottom"
        }
      }
    }
  ]
}
```

#### Simplified Mobile Dashboards

Create mobile-specific dashboard variants:
- **Fewer panels** per view
- **Larger text** and indicators
- **Essential metrics** only
- **Touch-friendly** controls

## 🎛️ Advanced Customization

### Custom Panel Plugins

#### Installing Plugins

```bash
# Install custom panel plugins
grafana-cli plugins install grafana-worldmap-panel
grafana-cli plugins install grafana-piechart-panel
```

#### Configuration Examples

```json
{
  "type": "worldmap",
  "options": {
    "showTooltip": true,
    "showLegend": true,
    "mapCenter": "North America",
    "mapCenterLatitude": 40,
    "mapCenterLongitude": -100
  }
}
```

### Custom Transformations

#### Data Processing

```json
{
  "transformations": [
    {
      "id": "calculateField",
      "options": {
        "mode": "reduceRow",
        "reduce": {
          "reducer": "sum"
        },
        "replaceFields": false,
        "alias": "Total Connections"
      }
    }
  ]
}
```

### Business Logic Integration

#### SLA Calculations

```flux
// Calculate uptime percentage
from(bucket: "telegraf")
  |> range(start: -30d)
  |> filter(fn: (r) => r["_measurement"] == "vpn_tunnel")
  |> filter(fn: (r) => r["_field"] == "Connected")
  |> map(fn: (r) => ({r with _value: if r._value == "true" then 1.0 else 0.0}))
  |> mean()
  |> map(fn: (r) => ({r with _value: r._value * 100.0}))
```

## 📁 Dashboard Management

### Version Control

#### Exporting Dashboards

```bash
# Export dashboard JSON for version control
curl -H "Authorization: Bearer $API_KEY" \
  "http://grafana:3000/api/dashboards/uid/vns3-system-overview" \
  | jq '.dashboard' > vns3-system-overview.json
```

#### Automated Deployment

```yaml
# Ansible playbook example
- name: Deploy VNS3 dashboards
  uri:
    url: "{{ grafana_url }}/api/dashboards/db"
    method: POST
    headers:
      Authorization: "Bearer {{ grafana_api_key }}"
      Content-Type: "application/json"
    body: "{{ lookup('file', item) | from_json }}"
    body_format: json
  loop:
    - vns3-system-overview.json
    - vns3-network-performance.json
```

### Environment Management

#### Multi-Environment Configurations

```json
{
  "templating": {
    "list": [
      {
        "name": "environment",
        "options": [
          {"text": "Production", "value": "prod"},
          {"text": "Staging", "value": "stage"},
          {"text": "Development", "value": "dev"}
        ]
      }
    ]
  }
}
```

## 🔍 Troubleshooting Custom Configurations

### Common Issues

**Variables not populating:**
- Check data source connectivity
- Verify query syntax
- Ensure data exists for time range

**Panels showing no data:**
- Validate field references in queries
- Check measurement and tag names
- Review time range compatibility

**Performance problems:**
- Optimize query aggregation windows
- Use appropriate time ranges
- Consider data retention policies

### Debugging Tools

```flux
// Add debugging to queries
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "snmp")
  |> yield(name: "debug_raw_data")  // See raw data
  |> filter(fn: (r) => r["hostname"] == "${hostname}")
  |> yield(name: "debug_filtered")  // See after filtering
```

---

**Next Steps:** See [Variables Reference](variables.md) for complete template variable documentation.