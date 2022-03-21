# Prometheus
_An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach._

## Prometheus Operator
Following CRDs are available right now:
- **`Prometheus`**, which defines a desired Prometheus deployment.
- **`Alertmanager`**, which defines a desired Alertmanager deployment.
- **`ThanosRuler`**, which defines a desired Thanos Ruler deployment.
- **`ServiceMonitor`**, which declaratively specifies how groups of Kubernetes services should be monitored. The Operator automatically generates Prometheus scrape configuration based on the current state of the objects in the API server.
- **`PodMonitor`**, which declaratively specifies how group of pods should be monitored. The Operator automatically generates Prometheus scrape configuration based on the current state of the objects in the API server.
- **`Probe`**, which declaratively specifies how groups of ingresses or static targets should be monitored. The Operator automatically generates Prometheus scrape configuration based on the definition.
- **`PrometheusRule`**, which defines a desired set of Prometheus alerting and/or recording rules. The Operator generates a rule file, which can be used by Prometheus instances.
- **`AlertmanagerConfig`**, which declaratively specifies subsections of the Alertmanager configuration, allowing routing of alerts to custom receivers, and setting inhibit rules.

> **NOTE:** Unfortunately there is no CRD for AlertManager silences right now ([GitHub Feature request](https://github.com/prometheus-operator/prometheus-operator/issues/2398)).



## Queries
### Troubleshooting Kubernetes
### Find Containers without configured Resources
`sum by (namespace,pod)(count by (namespace,pod,container)(kube_pod_container_info{container!="",namespace="<NAMESPACE>"}) unless sum by (namespace,pod,container)(kube_pod_container_resource_limits{resource="cpu"}))`

More information's: [Kubernetes resource limits](https://sysdig.com/blog/kubernetes-resource-limits/)



# Alertmanager
Unfortunately Google's chat webhooks are not compatible with Alertmanager's webhooks. But there is a community project called [calert](https://github.com/mr-karan/calert) that makes this possible. 

> **NOTE:** If you use *calert* the Alertmanager's receiver have to be named as the Google Chat name.