# Helm
_Helm helps you manage [Kubernetes](Kubernetes.md) applications — Helm Charts help you define, install, and upgrade even the most complex [Kubernetes](Kubernetes.md) application._


Get Helm Default Values (example for project "prometheus-community/kube-prometheus-stack"):
`helm show values prometheus-community/kube-prometheus-stack`

Testing Helm Chart (only local)
`helm template <RELEASE_NAME> .\<HELM-CHART_ROOT_LOCATION>`

Upgrade & Installing Helm Chart
`helm upgrade --install <RELEASE_NAME> -n <NAMESPACE> . \<HELM-CHART_ROOT_LOCATION>`

List Helm release 
`helm list --namespace test-ns1 `

Uninstall Helm release
`helm uninstall <RELEASE_NAME> -n <NAMESPACE>`


## Accessing a JSON-File inside Templates
The function to accessing a JSON-File inside a template (yaml) is **.Files.Get** and with indent you define how many withspaces helm uses.

> **Important:** You have to write _{{ .Files.Get...}}_ at the very beginning of the line!

**Example:**
```yaml
 ...
  json: |-
{{ .Files.Get "<YOUR_JSON_FILE>.json" | indent 4}}
```



## Range (foreach)
The *range* function is like a foreach function inside your Helm templates. 
In the following example we create a DaemonSet foreach Pre-Pull image defined in the Helm values *values.yaml*.

**Helm Template (daemonset.yaml)**
```yaml
{{- range .Values.prePullImages }}
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: {{ .containerName }}
spec:
  selector:
    matchLabels:
      name: {{ .containerName }}
  template:
    metadata:
      labels:
        name: {{ .containerName }}
    spec:
      containers:
      - name: {{ .containerName }}
        image: {{ .image }}
        imagePullPolicy: Always
        command: ["sleep", "3600"]
        resources:
          limits:
            cpu: 30m
            memory: 20Mi
          requests:
            cpu: 5m
            memory: 10Mi
{{- end }}
```


**Helm Values (values.yaml)**
```yaml
prePullImages:
  - containerName: <IMAGE_NAME>
    image: <IMAGE>
```


#### Use a Helm Value outside the "range"
To use a Helm value outside the range for example from the root values you have to use "$" before the values path like this:
`{{ $.Values.<YOUR_VALUENAME_AT_ROOT_PATH> }}`



## Troubleshooting
### Error: UPGRADE FAILED: rendered manifests contain a resource that already exists
**Error**
```
UPGRADE FAILED: rendered manifests contain a resource that already exists. Unable to continue with update: ConfigMap "<CONFIGMAP_NAME>" in namespace "<NAMESPACE>" exists and cannot be imported into the current release: invalid ownership metadata; label validation error: missing key "app.kubernetes.io/managed-by": must be set to "Helm"; annotation validation error: missing key "meta.helm.sh/release-name": must be set to "..."; annotation validation error: missing key "meta.helm.sh/release-namespace": must be set to "..."
```

**Cause**
*This error occurs because you already have manually deployed a resource with the same name (in this example a "ConfigMap") and you are trying to redeploy this with Helm again. Helm cannot override the ConfigMap because it was not deployed with Helm.*

**Solution**
Remove the resource, in this example the ConfigMap, and redeploy with Helm.