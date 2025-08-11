# Template Variables Reference

This document provides comprehensive documentation for all template variables used in VNS3 Grafana dashboards.

## 🔧 Core Variables

### Data Source Variable

**Name:** `DS_INFLUXDB`
**Type:** `datasource`
**Purpose:** Allows users to select the appropriate data source

```json
{
  "name": "DS_INFLUXDB",
  "type": "datasource",
  "query": "influxdb",
  "current": {
    "text": "InfluxDB",
    "value": "InfluxDB"
  },
  "hide": 0,
  "includeAll": false,
  "multi": false,
  "refresh": 1
}
```

**Usage in queries:**
```flux
from(bucket: "telegraf")
  // Automatically uses selected data source
```

### Hostname Variable

**Name:** `hostname`
**Type:** `query`
**Purpose:** Select specific VNS3 controller for monitoring

```json
{
  "name": "hostname",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"snmp\") |> keep(columns: [\"hostname\"]) |> distinct(column: \"hostnam
e\")",
  "current": {
    "text": "vns3-controller-01",
    "value": "vns3-controller-01"
  },
  "hide": 0,
  "includeAll": false,
  "multi": false,
  "refresh": 1,
  "sort": 0
}
```

**Usage in queries:**
```flux
|> filter(fn: (r) => r["hostname"] == "${hostname}")
```

**Expected values:**
- `vns3-us-east-1`
- `vns3-eu-west-1`
- `vns3-prod-01`
- Custom hostname from environment variables

### Topology Variable

**Name:** `topology`
**Type:** `query`
**Purpose:** Filter by VNS3 topology/environment

```json
{
  "name": "topology",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"snmp\") |> keep(columns: [\"customerPrefix\"]) |> distinct(column: \"c
ustomerPrefix\")",
  "current": {
    "text": "production",
    "value": "production"
  },
  "hide": 0,
  "includeAll": false,
  "multi": false,
  "refresh": 1,
  "sort": 0
}
```

**Usage in queries:**
```flux
|> filter(fn: (r) => r["customerPrefix"] == "${topology}")
// OR for API data
|> filter(fn: (r) => r["topology"] == "${topology}")
```

**Expected values:**
- `production`
- `staging`
- `development`
- `test`
- Custom topology names

## 🌐 Dashboard-Specific Variables

### IPsec Tunnel Variables

#### Instance Variable

**Name:** `instance`
**Type:** `query`
**Purpose:** Select specific IPsec tunnel for detailed analysis

```json
{
  "name": "instance",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"vpn_tunnel\") |> filter(fn: (r) => r.host == \"${hostname}\") |> keep(
columns: [\"instance\"]) |> distinct(column: \"instance\")",
  "multi": true,
  "includeAll": true,
  "allValue": ".*",
  "refresh": 1
}
```

**Usage:**
```flux
|> filter(fn: (r) => r["instance"] =~ /${instance:regex}/)
```

### Peer Variables

#### Peer Name Variable

**Name:** `peer_name`
**Type:** `query`
**Purpose:** Filter specific VNS3 peers

```json
{
  "name": "peer_name",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"vns3_peers\") |> filter(fn: (r) => r.host == \"${hostname}\") |> keep(
columns: [\"peer_name\"]) |> distinct(column: \"peer_name\")",
  "multi": true,
  "includeAll": true,
  "current": {
    "text": "All",
    "value": "$__all"
  }
}
```

#### Peer Type Variable

**Name:** `peer_type`
**Type:** `query`
**Purpose:** Filter by peer type (hub, spoke, mesh, etc.)

```json
{
  "name": "peer_type",
  "type": "custom",
  "options": [
    {"text": "All", "value": "$__all"},
    {"text": "Hub", "value": "hub"},
    {"text": "Spoke", "value": "spoke"},
    {"text": "Mesh", "value": "mesh"}
  ],
  "multi": true,
  "includeAll": true
}
```

### Client Pack Variables

#### Pack Name Variable

**Name:** `pack_name`
**Type:** `query`
**Purpose:** Select specific client packs

```json
{
  "name": "pack_name",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"vns3_clientpacks\") |> filter(fn: (r) => r.host == \"${hostname}\") |>
 keep(columns: [\"pack_name\"]) |> distinct(column: \"pack_name\")",
  "multi": true,
  "includeAll": true,
  "refresh": 1
}
```

## 🔍 Advanced Variable Configurations

### Chained Variables

Variables that depend on other variables for their options.

#### Topology → Hostname Chain

```json
{
  "name": "hostname_by_topology",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"snmp\") |> filter(fn: (r) => r.customerPrefix == \"${topology}\") |> k
eep(columns: [\"hostname\"]) |> distinct(column: \"hostname\")",
  "refresh": 2  // Refresh on dashboard load and time range change
}
```

#### Hostname → Tunnel Chain

```json
{
  "name": "tunnels_by_host",
  "type": "query",
  "datasource": "${DS_INFLUXDB}",
  "definition": "from(bucket: \"telegraf\") |> range(start: -1h) |> filter(fn: (r) => r._measurement == \"vpn_tunnel\") |> filter(fn: (r) => r.host == \"${hostname}\") |> keep(
columns: [\"instance\"]) |> distinct(column: \"instance\")",
  "refresh": 2
}
```

### Custom Variables

#### Environment-Based Variables

```json
{
  "name": "environment",
  "type": "custom",
  "options": [
    {"text": "Production", "value": "prod"},
    {"text": "Staging", "value": "stage"},
    {"text": "Development", "value": "dev"},
    {"text": "Test", "value": "test"}
  ],
  "current": {
    "text": "Production",
    "value": "prod"
  }
}
```

#### Time-Based Variables

```json
{
  "name": "aggregation_interval",
  "type": "interval",
  "auto": true,
  "auto_count": 30,
  "auto_min": "10s",
  "current": {
    "text": "auto",
    "value": "$__auto_interval_aggregation_interval"
  },
  "options": [
    {"text": "auto", "value": "$__auto_interval_aggregation_interval"},
    {"text": "30s", "value": "30s"},
    {"text": "1m", "value": "1m"},
    {"text": "5m", "value": "5m"},
    {"text": "15m", "value": "15m"}
  ]
}
```

**Usage:**
```flux
|> aggregateWindow(every: ${aggregation_interval}, fn: mean)
```

## 📊 Variable Usage Patterns

### Single Selection Patterns

```flux
// Exact match for single selection
|> filter(fn: (r) => r["hostname"] == "${hostname}")

// For dropdowns with "All" option
|> filter(fn: (r) => r["hostname"] =~ /${hostname}/)
```

### Multi-Selection Patterns

```flux
// Multi-select with regex
|> filter(fn: (r) => r["instance"] =~ /${instance:regex}/)

// Multi-select with pipe-separated values
|> filter(fn: (r) => r["peer_name"] =~ /${peer_name:pipe}/)

// Multi-select with exact matching
|> filter(fn: (r) => contains(value: r["pack_name"], set: [${pack_name:doublequote}]))
```

### Variable Formatters

Grafana provides built-in formatters for template variables:

```bash
${variable}           # Default: value1,value2,value3
${variable:regex}     # Regex: (value1|value2|value3)
${variable:pipe}      # Pipe: value1|value2|value3
${variable:csv}       # CSV: value1,value2,value3
${variable:doublequote} # Quoted: "value1","value2","value3"
${variable:singlequote} # Single quoted: 'value1','value2','value3'
${variable:sqlstring}   # SQL string: 'value1','value2','value3'
${variable:date}        # Date formatting
${variable:glob}        # Glob pattern: {value1,value2,value3}
```

## 🔄 Refresh Behaviors

### Refresh Options

```json
{
  "refresh": 0,  // Never refresh
  "refresh": 1,  // On dashboard load
  "refresh": 2   // On time range change
}
```

### Refresh Strategies

**Static variables (refresh: 0):**
- Environment names
- Fixed configuration options
- Rarely changing lists

**Load-time refresh (refresh: 1):**
- Hostnames (changes with infrastructure)
- Topology names
- Available data sources

**Dynamic refresh (refresh: 2):**
- Time-based selections
- Variables dependent on time range
- Frequently changing options

## 🎯 Best Practices

### Performance Optimization

#### Efficient Variable Queries

```flux
// Good - Specific time range and fields
from(bucket: "telegraf")
  |> range(start: -1h)
  |> filter(fn: (r) => r._measurement == "snmp")
  |> keep(columns: ["hostname"])
  |> distinct(column: "hostname")

// Avoid - Broad queries
from(bucket: "telegraf")
  |> range(start: -24h)
  |> distinct(column: "hostname")
```

#### Caching Strategies

- **Use appropriate refresh settings**
- **Limit time ranges for variable queries**
- **Cache frequently accessed variables**

### User Experience

#### Default Values

```json
{
  "current": {
    "text": "Most common value",
    "value": "most_common_value"
  }
}
```

#### Meaningful Labels

```json
{
  "options": [
    {"text": "Production Environment", "value": "prod"},
    {"text": "Staging Environment", "value": "stage"}
  ]
}
```

#### Logical Ordering

- **Alphabetical** for long lists
- **By importance** for operational priority
- **By frequency** for common selections

### Error Handling

#### Fallback Values

```flux
// Handle missing variables gracefully
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r._measurement == "snmp")
  |> filter(fn: (r) =>
    if exists r.hostname then
      r.hostname == "${hostname:default=all-controllers}"
    else
      true
  )
```

#### Validation Queries

```flux
// Check if hostname exists before using
hostname_exists = from(bucket: "telegraf")
  |> range(start: -1h)
  |> filter(fn: (r) => r.hostname == "${hostname}")
  |> count()
  |> findRecord(fn: (key) => true, idx: 0)
```

## 🔗 Integration Examples

### URL Parameter Passing

#### Dashboard Links with Variables

```html
<!-- Link to related dashboard with current context -->
<a href="/d/vns3-network-performance?var-hostname=${hostname}&var-topology=${topology}&from=${__from}&to=${__to}">
  View Network Performance
</a>
```

#### Deep Linking

```javascript
// JavaScript for custom buttons
window.location.href = `/d/vns3-ipsec-tunnels?var-hostname=${hostname}&var-instance=${selectedTunnel}`;
```

### API Integration

#### Programmatic Variable Updates

```bash
# Update dashboard variables via API
curl -X POST \
  -H "Authorization: Bearer $API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"variables": {"hostname": {"current": {"text": "new-host", "value": "new-host"}}}}' \
  "$GRAFANA_URL/api/dashboards/uid/vns3-system-overview"
```

## 🚨 Troubleshooting

### Common Issues

**Variable not populating:**
- Check data source connection
- Verify query syntax and time range
- Ensure data exists for the query period

**Variables not cascading:**
- Check refresh settings on dependent variables
- Verify query dependencies are correct
- Review filter syntax in chained queries

**Performance problems:**
- Reduce query time ranges
- Use more specific filters
- Consider caching frequently used variables

### Debug Techniques

#### Variable Query Testing

1. **Copy variable query** to data source query editor
2. **Test with different time ranges**
3. **Check for syntax errors**
4. **Verify expected data format**

#### Browser Network Tab

- **Monitor variable query requests**
- **Check response times and sizes**
- **Identify slow-loading variables**

---

**Related Documentation:**
- [Installation Guide](installation.md) - Setting up template variables
- [Customization Guide](customization.md) - Advanced variable configuration