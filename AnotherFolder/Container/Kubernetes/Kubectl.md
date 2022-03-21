# Kubectl
_The [Kubernetes](Kubernetes.md) command-line tool, [Kubectl](Kubectl.md) allows you to run commands against Kubernetes clusters. You can use kubectl to deploy applications, inspect and manage cluster resources, and view logs._

## Basic Commands
Get configured contexts: `kubectl config get-contexts`
Switch context: `kubectl config use-context <NAME>`
Get pods: `kubectl get pods -n <NAMESPACE>`
Kill pod: `kubectl delete pod -n <NAMESPACE> <POD_NAME> --force --grace-period=0
Get more details (for example of a pod): `kubectl get pod -n <NAMESPACE> <POD_NAME> -o wide`
Describe pod: `kubectl describe pod -n <NAMESPACE> <POD_NAME>`
Get persistent volumes: `kubectl get pvc`
Get pods filtered on label: `kubectl get pods --all-namespaces  -l app=nginx`
Open container shell: `kubectl exec -n <NAMESPACE> --stdin --tty <POD_NAME> -- /bin/sh`
Sort by (for example pod names): `kubectl get pods --all-namespaces --sort-by=.metadata.name`
Sort by (for example persistent volumes): `kubectl get pods --all-namespaces --sort-by=.spec.capacity.storage`
Sort by (for example events): `kubectl get events --sort-by=.metadata.creationTimestamp`
Get all supported resource types: `kubectl api-resources`
`
Create file and apply (secret example)
_Because `kubectl create` can only be run once, you can output the secret as YAML and pipe this to `kubectl apply`._
`kubectl create secret generic <SECRET_NAME> -n <NAMESPACE> --from-literal=<USERNAME_VARIABLE>=<USERNAME> --from-literal=<PASSWORD_VARIABLE>=<PASSWORD> --dry-run=client -o yaml | kubectl apply -f -`

-> More in the [Cheatsheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)


## Namespaces
Get all namespaces: `kubectl get namespace`
Create namespace: `kubectl create namespace <NAMESPACE_NAME>`

Create namespace (example YAML): 
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: <NAMESPACE>
```
-> Apply namespace: `kubectl apply -f .\namespace.yml`



## Examples
### Deployment of a basic Nginx Server
deployment.yaml:
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
 name: <DEPLOYMENT_NAME>
 labels:
   app: nginx
spec:
 replicas: 3
 selector:
   matchLabels:
     app: nginx
 template:
   metadata:
     labels:
       app: nginx
   spec:
     containers:
     - name: nginx
       image: nginx:1.14.2
       ports:
       - containerPort: 80
```
-> Apply deployment: `kubectl apply -f .\deployment.yml -n <NAMESPACE>`


service.yaml:
```yaml
apiVersion: v1
kind: Service
metadata:
 name: <SERVICE_NAME>
spec:
 type: ClusterIP
 selector:
   app: nginx
 ports:
 # By default and for convenience, the `targetPort` is set to the same value as the `port` field.
 - port: 80
   targetPort: 80
   # Optional field
   # By default and for convenience, the Kubernetes control plane will allocate a port from a range (default: 30000-32767)
   # nodePort: 30011
```
-> Apply service: `kubectl apply -f .\service.yml -n <NAMESPACE>`


ingress.yaml:
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
 name: <INGRESS_NAME>
spec:
 rules:
 - host: nginx.apps.yourdomain.ch
 http:
   paths:
   - path: /
     pathType: Prefix
     backend:
       service:
         name: <SERVICE_NAME>
         port:
           number: 80
```
-> Apply ingress: `kubectl apply -f .\ingress.yaml -n <NAMESPACE>`



## Windows: Autocompletion
Add [Kubectl autocompletion](https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/#enable-shell-autocompletion) to load on each PowerShell session:
````powershell
kubectl completion powershell >> $PROFILE
````



## Troubleshooting Kubernetes
### Kubectl: Get Containers Resource Configuration (find Containers without configured Resources)
**JSONPATH**
`kubectl get pods -n <NAMESPACE> -o jsonpath="{range .items[*]}{.metadata.name}{'\t'}{.spec.containers[*].resources}{'\n'}{end}"`

**Custom Columns**
`kubectl get pods -n grafana --output=custom-columns="POD:.metadata.name,CONTAINERS:.spec.containers[*].name,REQUESTS:.spec.containers[*].resources.requests,LIMITS:.spec.containers[*].resources.limits"`