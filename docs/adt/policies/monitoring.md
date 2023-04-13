# Monitoring Policies

Metric collection is disabled by default. The node exporter for  can be enabled through the monitoring
policy below. If the policy is omitted, or if one property is left undefined,
then the relevant metric collection will be disabled.

```yaml
policies:
- monitoring:
    type: tosca.policies.Monitoring.MiCADO
    properties:
      enable_container_metrics: true
      enable_node_metrics: true
```