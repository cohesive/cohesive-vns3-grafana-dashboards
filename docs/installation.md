# Installation Guide

This guide walks you through setting up VNS3 Grafana dashboards from initial VNS3 configuration to fully functional monitoring.

## 📋 Prerequisites

### Required Components

1. **VNS3 Controller** (any supported version)
2. **Grafana** 7.0 or higher
3. **Time Series Database** (InfluxDB, Prometheus, or compatible)
4. **Telegraf** (installed via VNS3 Plugin)

### Network Requirements

- **VNS3 Controller** accessible for SNMP (port 161) and API calls
- **Grafana** can connect to your time series database
- **Internet access** for VNS3 plugin installation (or private catalog)

## 🚀 Step 1: Install VNS3 Telegraf Plugin

### From Public Catalog

1. **Navigate** to VNS3 Controller → **Plugins** → **Catalog**
2. **Find** "Telegraf" in the public catalog
3. **Click** "Install" to add plugin image to controller
4. **Wait** for installation to complete

### From Private Catalog or Upload

1. **Configure** private plugin catalog (if applicable)
2. **Upload** plugin image directly via VNS3 UI
3. **Verify** plugin appears in "Ready" status

## 🔧 Step 2: Configure VNS3 Telegraf Plugin

### Environment Variables

Create `/etc/default/telegraf` configuration:

```bash
# Data collection controls
ENABLE_SNMP=true
ENABLE_IPSEC=true
ENABLE_API=true

# VNS3 identification
HOSTNAME=vns3-us-east-1
TOPOLOGY=production

# VNS3 connection details
VNS3_IP=10.0.1.1
API_TOKEN=your-api-token-here
```

### API Token Setup

1. **Navigate** to VNS3 Controller → **Access Management**
2. **Click** "New API Token"
3. **Configure:**
   - **Name:** "Telegraf Monitoring"
   - **Lifetime:** 1 hour (with refresh)
   - **Check:** "Token lifetime refreshes when used"
4. **Copy** token for environment variables

### Start Plugin Instance

1. **Navigate** to **Plugins** → **Ready** → **Telegraf**
2. **Click** Actions → **"Start Instance"**
3. **Configure:**
   - **Name:** telegraf-monitor
   - **Description:** VNS3 monitoring collector
   - **IP Address:** Auto-assign or specify
4. **Click** "Start"

## 🌐 Step 3: Configure VNS3 Firewall

### Required Firewall Rules

Add these rules to your VNS3 firewall:

```bash
# Allow Telegraf to access VNS3 SNMP
INPUT -p udp -s <telegraf-plugin-ip> --dport 161 -j ACCEPT

# Allow VNS3 to respond to Telegraf
OUTPUT -d <telegraf-plugin-ip> -j ACCEPT

# Allow Telegraf traffic through VNS3
FORWARD -s <telegraf-plugin-ip> -j ACCEPT
FORWARD -d <telegraf-plugin-ip> -j ACCEPT

# For push-based outputs (InfluxDB, etc.)
POSTROUTING -o eth0 -s <telegraf-plugin-ip> -j MASQUERADE-ONCE

# For pull-based outputs (Prometheus, etc.)
PREROUTING -p tcp --dport 9273 -d ${vns3_primary_private_ip} -j DNAT --to <telegraf-plugin-ip>:9273
```

Replace `<telegraf-plugin-ip>` with your actual plugin IP address.

## 💾 Step 4: Configure Time Series Database

### InfluxDB v2 Example

```yaml
# influxdb.conf
[[outputs.influxdb_v2]]
  urls = ["http://your-influxdb:8086"]
  token = "your-influxdb-token"
  organization = "your-org"
  bucket = "telegraf"
```

### Prometheus Example

```yaml
# prometheus.yml
[[outputs.prometheus_client]]
  listen = ":9273"
  metric_version = 2
```

### Update Telegraf Configuration

1. **Edit** telegraf.conf via Plugin Manager
2. **Add** appropriate output configuration
3. **Restart** Telegraf service

## 📊 Step 5: Install Grafana Dashboards

### Method 1: Import JSON Files

1. **Download** dashboard JSON files from this repository
2. **Open** Grafana → **Dashboards** → **Import**
3. **Upload** JSON file or paste content
4. **Configure:**
   - **Data Source:** Select your configured data source
   - **Folder:** Choose appropriate folder
5. **Click** "Import"

### Method 2: Grafana CLI (if available)

```bash
# Download dashboard files
curl -O https://raw.githubusercontent.com/cohesive/vns3-grafana-dashboards/main/dashboards/vns3-system-overview.json

# Import using grafana-cli (if installed)
grafana-cli admin import vns3-system-overview.json
```

## ⚙️ Step 6: Configure Data Source

### InfluxDB Data Source

1. **Navigate** to Grafana → **Configuration** → **Data Sources**
2. **Click** "Add data source"
3. **Select** "InfluxDB"
4. **Configure:**
   - **URL:** http://your-influxdb:8086
   - **Database:** telegraf (v1) or configure v2 settings
   - **User/Password:** As configured
5. **Test** connection and save

### Prometheus Data Source

1. **Add** Prometheus data source
2. **Configure:**
   - **URL:** http://telegraf-plugin-ip:9273
   - **Scrape interval:** 30s
3. **Test** and save

## 🔧 Step 7: Verify Installation

### Check Data Collection

1. **Monitor** Telegraf logs via Plugin Manager
2. **Verify** metrics in your time series database
3. **Test** simple queries

### Dashboard Verification

1. **Open** each imported dashboard
2. **Check** that panels display data
3. **Verify** template variables work
4. **Test** time range selection

### Common Issues

**No data showing:**
- Verify Telegraf is running and collecting metrics
- Check data source connection in Grafana
- Confirm SNMP and API access from plugin
- Review firewall rules

**Template variables empty:**
- Ensure data exists in time series database
- Check variable queries match your data structure
- Verify hostname/topology tags are present

## 📈 Step 8: Customize for Your Environment

### Template Variables

Update dashboard variables to match your setup:

1. **Edit** dashboard → **Settings** → **Variables**
2. **Modify** queries to match your naming conventions
3. **Test** variable population

### Panel Customization

- **Adjust thresholds** for your environment
- **Modify colors** and styling preferences
- **Add custom panels** for additional metrics

### Alerting (Optional)

1. **Configure** notification channels
2. **Set up alerts** on critical panels
3. **Test** alert delivery

## 🔄 Step 9: Maintenance

### Regular Tasks

- **Monitor** Telegraf plugin health
- **Refresh** API tokens before expiration
- **Update** dashboards with new releases
- **Review** and optimize queries

### Backup

- **Export** customized dashboards regularly
- **Document** custom configurations
- **Version control** dashboard JSON files

## ❓ Troubleshooting

### Data Collection Issues

```bash
# Check Telegraf logs
docker logs telegraf-container

# Test SNMP connectivity
snmpwalk -v2c -c vns3public <vns3-ip> system

# Verify API access
curl -H "Authorization: Bearer <token>" http://<vns3-ip>/api/status
```

### Dashboard Issues

1. **Check** Grafana logs for errors
2. **Verify** data source connectivity
3. **Test** queries in data source query editor
4. **Validate** JSON syntax if manually edited

### Performance Optimization

- **Use** appropriate time ranges
- **Optimize** query windows
- **Consider** data retention policies
- **Monitor** dashboard load times

## 📞 Support

- **GitHub Issues:** Technical problems and bugs
- **Cohesive Support:** VNS3-specific configuration help
- **Community Forum:** General questions and sharing

---

**Next Steps:** See [Customization Guide](customization.md) for advanced configuration options.