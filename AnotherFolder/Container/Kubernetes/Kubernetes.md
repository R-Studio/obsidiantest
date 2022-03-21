# Kubernetes
Rancher Kubernetes is using Docker as container runtime/engine.

Kubernetes internal communication): 
`<KUBERNETES_SERVICE_NAME>:<APPLICATION_PORT>`

Kubernetes internal communication (pods in different namespaces): 
`<KUBERNETES_SERVICE_NAME>.<KUBERNETES_NAMESPACE>:<APPLICATION_PORT>`


## Resources
### [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/)
_A Secret is an object that contains a small amount of sensitive data such as a password, a token, or a key. Such information might otherwise be put in a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/) specification or in a [container image](https://kubernetes.io/docs/reference/glossary/?all=true#term-image). Using a Secret means that you don't need to include confidential data in your application code._

#### Creating Secrets
**Kubectl (example for USERNAME and PASSWORD)**
```console
kubectl create secret generic <SECRET_NAME> -n <NAMESPACE> --from-literal=<USERNAME_VARIABLE>=<USERNAME> --from-literal=<PASSWORD_VARIABLE>=<PASSWORD>
```

**YAML**
```yaml
apiVersion: v1
kind: Secret
metadata:
 name: <SECRET_NAME>
stringData:
 <USERNAME_VARIABLE>: <USERNAME>
 <PASSWORD_VARIABLE>: <PASSWORD>
```

[Useful article](https://medium.com/avmconsulting-blog/secrets-management-in-kubernetes-378cbf8171d0#:~:text=Kubernetes%20Secrets%20are%20secure%20objects%20which%20store%20sensitive%20data%2C%20such,how%20sensitive%20data%20is%20used) 




## Resource Management
By default, containers run with unbounded [compute resources](https://kubernetes.io/docs/concepts/configuration/manage-resources-containers/) on a Kubernetes cluster. With resource quotas, cluster administrators can restrict resource consumption and creation on a [namespace](https://kubernetes.io/docs/concepts/overview/working-with-objects/namespaces) basis. Within a namespace, a Pod or Container can consume as much CPU and memory as defined by the namespace's resource quota or by its limits. 
-> [Basic Resource Management](https://www.puzzle.ch/de/blog/articles/2021/11/04/basic-kubernetes-resource-management)

### Requests & Limits (Containers)
*When you specify a [Pod](https://kubernetes.io/docs/concepts/workloads/pods/), you can optionally specify how much of each resource a [container](https://kubernetes.io/docs/concepts/containers/) needs. 
- ***Requests:** When you specify the resource **request** for containers in a Pod, the [kube-scheduler](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-scheduler/) uses this information to decide **which node to place the Pod on and reserves** at least the **request** amount of that system resource specifically for that container to use.* 
- ***Limits:** When you specify a resource **limit** for a container, the kubelet **enforces those limits so that the running container is not allowed to use more** of that resource than the limit you set.*

> **Note:** If a container specifies its own cpu/memory limit, but does not specify a cpu/memory request, Kubernetes automatically assigns a cpu/memory request that matches the limit. 

#### Example: Request & Limit
```yaml
...
    resources:
      requests:
        cpu: "100m"
        memory: "50Mi"
      limits:
        cpu: "1500m"
        memory: "500Mi"
```


### Compute Resource Quotas (Namespace)
*A **resource quota**, defined by a `ResourceQuota` object, provides constraints that **limit aggregate resource consumption (REQUESTS & LIMITS) per namespace**. It can limit the quantity of objects that can be created in a namespace by type, as well as the total amount of compute resources that may be requested or limited by resources in that namespace.*


### Limit Ranges
*A LimitRange is a policy to constrain resource allocations (to Pods or Containers) in a namespace and can provides constraints that can:*
- Enforce minimum and maximum compute resources usage per Pod or Container in a namespace.
- Enforce minimum and maximum storage request per PersistentVolumeClaim in a namespace.
- Enforce a ratio between request and limit for a resource in a namespace.
- Set default request/limit for compute resources in a namespace and automatically inject them to Containers at runtime.

#### Example Limit Range
```yaml
apiVersion: v1
kind: LimitRange
metadata:
  name: default-limit-range
spec:
  limits:
  - default:
      cpu: 300m
      memory: 20Mi
    defaultRequest:
      cpu: 50m
      memory: 10Mi
    type: Container
```



### Known-Errors
**Error:** *Create Pod ... failed error: pods "..." is forbidden: failed quota: <YOUR_QUOTA>: must specify limits.cpu,limits.memory*
**Solution:** Add limits to your container deployment.

**Error:** *Create Pod ... failed error: pods "..." is forbidden: exceeded quota: <YOUR_QUOTA>, requested: limits.cpu=1600m, used: limits.cpu=1, limited: limits.cpu=2500m*
**Solution:** In this example my quota is exceeded (`limited - (requested + used) = -100m`). -> Increase your quota by at least 100m.


### Vertical Pod Autoscaler
- [ ] Doku Vertical Pod Autoscaler #todo 





## Persistent Volumes
[Benchmarking Kubernetes Storage Solutions](https://www.vshn.ch/blog/benchmarking-kubernetes-storage-solutions/)

### Longhorn
[_Longhorn](https://github.com/longhorn/longhorn) delivers simplified, easy to deploy and upgrade, 100% open source, cloud-native hyper-converged persistent block storage without the cost overhead of open core or proprietary alternatives._

Project source code is spread across a number of repos:
| Component                      | What it does                                                           | GitHub repo                                                                                 |
|:------------------------------ |:---------------------------------------------------------------------- |:------------------------------------------------------------------------------------------- |
| Longhorn Backing Image Manager | Backing image download, sync, and deletion in a disk                   | [longhorn/backing-image-manager](https://github.com/longhorn/backing-image-manager)         |
| Longhorn Engine                | Core controller/replica logic                                          | [longhorn/longhorn-engine](https://github.com/longhorn/longhorn-engine)                     |
| Longhorn Instance Manager      | Controller/replica instance lifecycle management                       | [longhorn/longhorn-instance-manager](https://github.com/longhorn/longhorn-instance-manager) |
| Longhorn Manager               | Longhorn orchestration, includes CSI driver for Kubernetes             | [longhorn/longhorn-manager](https://github.com/longhorn/longhorn-manager)                   |
| Longhorn Share Manager         | NFS provisioner that exposes Longhorn volumes as ReadWriteMany volumes | [longhorn/longhorn-share-manager](https://github.com/longhorn/longhorn-share-manager)       |
| Longhorn UI                    | The Longhorn dashboard                                                 | [longhorn/longhorn-ui](https://github.com/longhorn/longhorn-ui)  


#### Creating Longhorn Persistent Volumes using Rancher
- Create Project (I named it *Storage*)
- In Rancher GUI: Cluster Tools -> Longhorn -> Install (default settings)
- *If you want: 
	- changing storage reserve (Longhorn GUI -> Node -> Edit node and disks)*
	- changing the storage overprovisioning percentage to 200% (Longhorn GUI -> Setting -> General -> Stroage Over...)
- Now you have a new StorageClass called *longhorn* and it is ready to use.


#### Performance
Currently there is a big discussion about Longhorns performance, more details in the [GitHub Issue](https://github.com/longhorn/longhorn/issues/1541).
*-> Maybe this will be better in the future*

#### Monitoring
[Monitoring Longhorn](https://longhorn.io/docs/latest/monitoring/prometheus-and-grafana-setup/) is simple. All you need is:
- A running Prometheus instance
- A Kubernetes ServiceMonitor for Longhorn

**Example: ServiceMonitor for Longhorn**
Longhorn ServiceMonitor has a label selector `app: longhorn-manager` to select Longhorn backend service. Later on, the Prometheus CRD can include Longhorn ServiceMonitor so that the Prometheus server can discover all Longhorn manager pods and their endpoints. Deploy the ServiceMonitor in the same namespace as Prometheus.
```yaml
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  name: longhorn-prometheus-servicemonitor
  labels:
    name: longhorn-prometheus-servicemonitor
spec:
  selector:
    matchLabels:
      app: longhorn-manager
  namespaceSelector:
    matchNames:
      - longhorn-system
  endpoints:
    - port: manager
```

**IMPORTANT:** Unfortunately in my case the Prometheus Operator has not automatically found the new created ServiceMonitor. 
So I had to disable serviceMonitorSelectorNilUsesHelmValues and podMonitorSelectorNilUsesHelmValues like this (more information's in the [GitHub issue](https://github.com/helm/charts/issues/13196)):
```yaml
prometheus:
  prometheusSpec:
    serviceMonitorSelectorNilUsesHelmValues: false
    podMonitorSelectorNilUsesHelmValues: false
```

Longhorn also offers a [Grafana Dashboard](https://grafana.com/grafana/dashboards/13032) (ID: 13032) that can be imported.



### Rook
*[Rook](https://rook.io/) is a open-source Cloud-Native storage for Kubernetes that is a production ready management for File, Block and Object Storage by using Ceph. There are other storage providers like [Cassandra](https://github.com/k8ssandra/cass-operator/) and [NFS](https://github.com/rook/nfs) (Alpha). Rook turns distributed storage systems into self-managing, self-scaling, self-healing storage services. It automates the tasks of a storage administrator: deployment, bootstrapping, configuration, provisioning, scaling, upgrading, migration, disaster recovery, monitoring, and resource management.*



### Piraeus
https://itnext.io/state-of-persistent-storage-in-k8s-a-benchmark-77a96bb1ac29 

[Piraeus-Project](https://github.com/piraeusdatastore/piraeus)
[Piraeus-Operator](https://github.com/piraeusdatastore/piraeus-operator)




## Taints & Tolerations
*[Taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) and [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) are a mechanism that allows you to ensure that pods are not placed on inappropriate nodes. **[Taints](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) are added to nodes, while [tolerations](https://kubernetes.io/docs/concepts/scheduling-eviction/taint-and-toleration/) are defined in the pod specification.** When you taint a node, it will repel all the pods except those that have a toleration for that taint. A node can have one or many taints associated with it.
For example, most Kubernetes distributions will automatically taint the master nodes so that one of the pods that manages the control plane is scheduled onto them and not any other data plane pods deployed by users. This ensures that the master nodes are dedicated to run control plane pods.* -> **[Useful Guide](https://www.densify.com/kubernetes-autoscaling/kubernetes-taints)**

### Taint Effects
| Effect           | Description                                                                                                     |
| ---------------- | --------------------------------------------------------------------------------------------------------------- |
| NoSchedule       | The Kubernetes scheduler will only **allow** scheduling **pods that have tolerations** for the tainted nodes.           |
| PreferNoSchedule | The Kubernetes scheduler will try to **avoid** scheduling **pods that don’t have tolerations** for the tainted nodes.   |
| NoExecute        | Kubernetes will **evict the running pods** from the nodes if the pods **don’t have tolerations** for the tainted nodes. |

### Taint Operations
The default value for `operator` is `Equal`. A toleration "matches" a taint if the keys are the same and the effects are the same, and:
- the `operator` is `Exists` (in which case no `value` should be specified), or
- the `operator` is `Equal` and the `value`s are equal.
> **NOTE:** There are two special cases:
An empty `key` with operator `Exists` matches all keys, values and effects which means this will tolerate everything.
An empty `effect` matches all effects with key `key1`.


### Commands
Get taints of all nodes: 
```console
kubectl get nodes -o=custom-columns=NodeName:.metadata.name,TaintKey:.spec.taints[*].key,TaintValue:.spec.taints[*].value,TaintEffect:.spec.taints[*].effect
```


### Example: Deploy Pod on Controlplane/etcd Nodes
For example you want to monitor your Kubernetes cluster using *Prometheus Node Exporter*, but your *Prometheus Node Exporter* pods are only deployed on the worker nodes (w01, w02 & w03). This is because you forgot define tolerations to your pods specification. To add this toleration to your pods, first check your nodes taint configurations. The following is an example:
```console
NodeName   TaintKey                                                            TaintValue   TaintEffect
cp01       node-role.kubernetes.io/controlplane,node-role.kubernetes.io/etcd   true,true    NoSchedule,NoExecute
w01        <none>                                                              <none>       <none>
w02        <none>                                                              <none>       <none>
w03        <none>                                                              <none>       <none>
```

Now you know the taints and can configure the tolerations. Because I used the *Prometheus-Operator Helm chart* to deploy all *Prometheus Node Exporter* I had to add following in my Helm values:
```yaml
prometheus-node-exporter:
  tolerations:
    - key: node-role.kubernetes.io/controlplane
      operator: "Exists"
      effect: NoSchedule
    - key: node-role.kubernetes.io/etcd
      operator: "Exists"
      effect: NoExecute
```
*-> Normally this tolerations are defined in `spec.tolerations:`*


### Tolerating all Taints
You can configure a pod to tolerate all taints by adding an `operator: "Exists"` toleration with no `key` and `value` parameters. Pods with this toleration are not removed from a node that has taints.
```yaml
...
  tolerations: 
    - operator: "Exists"
```

### Default Tolerations on Namespace
There is a feature called [*PodTolerationRestriction*](https://kubernetes.io/docs/reference/access-authn-authz/admission-controllers/#podtolerationrestriction) to define default tolerations on namespace level. Example for namespace annotations:
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: apps-that-need-nodes-exclusively
  annotations:
    scheduler.alpha.kubernetes.io/defaultTolerations: '[{"operator": "Exists", "effect": "NoSchedule", "key": "dedicated-node"}]'
    scheduler.alpha.kubernetes.io/tolerationsWhitelist: '[{"operator": "Exists", "effect": "NoSchedule", "key": "dedicated-node"}]'
```




# Security
## Network Policies
- [ ] Dokumenttation Network Policies #todo 




## Kubernetes Tools
### Kubernetes local (Testing purposes)
Recommended: https://rancherdesktop.io/


### Kaniko
With Kaniko you are able to build container images on Kubernetes without root or privileged mode.




## Troubleshooting
- [Kubectl: Get Containers Resource Configuration](Kubectl.md#Kubectl%20Get%20Containers%20Resource%20Configuration%20find%20Containers%20without%20configured%20Resources) / [Prometheus: Find Containers without configured Resources](../../Monitoring/Prometheus.md#Find%20Containers%20without%20configured%20Resources%20limits)




### Prometheus: [Find Containers without configured Resources (limits)](../../Monitoring/Prometheus.md#Find%20Containers%20without%20configured%20Resources%20limits)






