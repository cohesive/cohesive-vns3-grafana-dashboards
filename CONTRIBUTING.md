# Contributing to VNS3 Grafana Dashboards

Thank you for your interest in improving VNS3 monitoring dashboards! This document provides guidelines for contributing to the project.

## 🤝 How to Contribute

### Reporting Issues

1. **Check existing issues** first to avoid duplicates
2. **Use issue templates** when available
3. **Provide detailed information:**
   - VNS3 version and configuration
   - Grafana version
   - Dashboard JSON version
   - Expected vs actual behavior
   - Screenshots if applicable

### Suggesting Enhancements

We welcome suggestions for:
- **New dashboard panels** and visualizations
- **Additional metrics** from VNS3 API or SNMP
- **Performance improvements** to queries
- **Visual enhancements** and UX improvements
- **Documentation improvements**

## 🔧 Development Guidelines

### Dashboard Modifications

1. **Test thoroughly** with real VNS3 data
2. **Follow existing patterns** for consistency
3. **Use template variables** for multi-controller support
4. **Document query logic** in panel descriptions
5. **Optimize queries** for performance

### Query Best Practices

```flux
// Good - Efficient filtering
from(bucket: "telegraf")
  |> range(start: v.timeRangeStart, stop: v.timeRangeStop)
  |> filter(fn: (r) => r["_measurement"] == "snmp")
  |> filter(fn: (r) => r["hostname"] == "${hostname}")
  |> aggregateWindow(every: v.windowPeriod, fn: mean)

// Avoid - Inefficient broad queries
from(bucket: "telegraf")
  |> range(start: -1d)
  |> filter(fn: (r) => r["_measurement"] =~ /.*/)
```

### Visual Standards

- **Use consistent colors** across dashboards
- **Set appropriate thresholds** for alerts
- **Include units** for all metrics
- **Use descriptive panel titles**
- **Add hover tooltips** where helpful

## 📋 Submission Process

### Pull Request Workflow

1. **Fork the repository**
2. **Create feature branch:**
   ```bash
   git checkout -b feature/new-dashboard-panel
   ```
3. **Make changes following guidelines**
4. **Test with multiple VNS3 configurations**
5. **Update documentation** if needed
6. **Submit pull request** with clear description

### Pull Request Requirements

- [ ] **Descriptive title** and detailed description
- [ ] **Testing completed** on real VNS3 environment
- [ ] **Documentation updated** if applicable
- [ ] **Screenshots included** for visual changes
- [ ] **JSON validates** properly in Grafana
- [ ] **Variables work** across different topologies

## 🧪 Testing

### Local Testing Setup

1. **VNS3 Controller** with Telegraf plugin
2. **Grafana instance** with test data source
3. **Sample data** for edge cases

### Test Scenarios

- **Single controller** deployment
- **Multi-controller** with different topologies
- **Various VNS3 configurations:**
  - Different numbers of IPsec tunnels
  - Multiple peer types
  - Various client pack sizes
  - Mixed enabled/disabled states

### Testing Checklist

- [ ] Dashboard imports without errors
- [ ] All panels display data correctly
- [ ] Template variables populate properly
- [ ] Time range selection works
- [ ] Refresh functionality operates
- [ ] Mobile/tablet responsiveness
- [ ] Performance acceptable with large datasets

## 📝 Documentation Standards

### Code Comments

```json
{
  "title": "CPU Usage",
  "targets": [{
    "query": "// CPU utilization from SNMP data\n// Aggregated by time window for performance\nfrom(bucket: \"telegraf\")..."
  }]
}
```

### Panel Descriptions

Include helpful descriptions in dashboard panels:
- **What the metric represents**
- **Normal value ranges**
- **When to investigate anomalies**
- **Related panels or dashboards**

## 🏷️ Commit Guidelines

### Commit Message Format

```
type(scope): brief description

Longer explanation if needed
- Additional bullet points
- More context about the change

Fixes #123
```

### Commit Types

- **feat:** New dashboard or panel
- **fix:** Bug fix or correction
- **docs:** Documentation changes
- **style:** Visual/formatting changes
- **refactor:** Code restructuring
- **perf:** Performance improvements
- **test:** Testing additions

### Examples

```bash
feat(dashboard): add IPsec tunnel latency monitoring

- New panel showing tunnel RTT metrics
- Threshold alerts for high latency
- Compatible with VNS3 v5.x API

Fixes #45

fix(query): correct memory utilization calculation

Memory percentage was showing inverted values due to
incorrect formula in InfluxDB query.

style(panels): improve color consistency across dashboards
```

## 📚 Resources

### Useful Links

- **[Grafana Documentation](https://grafana.com/docs/)**
- **[InfluxDB Flux Language](https://docs.influxdata.com/influxdb/v2/query-data/flux/)**
- **[VNS3 API Documentation](https://docs.cohesive.net/)**
- **[Telegraf Input Plugins](https://docs.influxdata.com/telegraf/v1/plugins/#input-plugins)**

### Getting Help

- **GitHub Issues:** Technical questions and bugs
- **Discussions:** General questions and ideas
- **Cohesive Support:** VNS3-specific issues

## 📄 License

By contributing, you agree that your contributions will be licensed under the same Apache 2.0 License that covers the project.

## 🙏 Recognition

Contributors will be acknowledged in the project README and release notes. Thank you for helping make VNS3 monitoring better for everyone!

---

**Questions?** Open an issue or start a discussion. We're here to help!