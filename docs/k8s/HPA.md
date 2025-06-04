# HPA K8s Documentation

How to setup HPA in K8s Cluster

## Workspace

First you need a metric-server for collect you Kubelets metrics expose to Kubernetes API Server.

### Installation:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Validation:
```bash
kubectl top nodes
kubectl top pods
# se estiver tudo normal e o metric server estiver funcionando ele vai aparecer as metricas
```

Default configuration for metric-server:
```bash
kubectl edit deployment metrics-server -n kube-system

# dentro do arquivo procure por args e adicione essas linhas:
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

---

## HPA Manifest
```yaml
apiVersion: autoscaling/v2 # tem que ser v2
kind: HorizontalPodAutoscaler
metadata:
  name: monitor-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Values.deployment.name }} # so o nome do deployment indicado no values helm
		# nesse caso usei para o deployment
  minReplicas: 2 
  maxReplicas: 5
  metrics:
  - type: Resource
    resource:
      name: cpu
      target:
        type: Utilization
        averageUtilization: 50 # porcentagem da utilzacao para escalar
  - type: Resource
    resource:
      name: memory
      target:
        type: Utilization
        averageUtilization: 70 # porcentagem da utilzacao para escalar
```

HPA Visualization:
```bash
kubectl get hpa # pode colocar um -w para fazer com que o hpa vai ficar executando ate mudar alguma coisa 
```

## Deployment configuration

Be very careful with the indentation. If there's an error inside the HPA but the 'top nodes' command works, check if the indentation is correct. A common error is showing 'unknown' in the HPA while the metrics are working—this usually means the indentation is wrong in the deployment or in the specified configuration.
```yaml
    spec:
      containers:
      - name: app
        image: {{ .Values.image }}
        ports:
        - containerPort: 5000
        resources: # tomar cuidado com a identacao dessa merda
          requests:
            cpu: "100m"
            memory: "64Mi"
          limits:
            cpu: "200m" # nao e bom definir um limite de cpu mas coloquei aqui para teste
            memory: "128Mi"
```