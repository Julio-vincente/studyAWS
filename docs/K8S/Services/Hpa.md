# HPA

## Referências

* [Metrics Server](https://github.com/kubernetes-sigs/metrics-server)
* [Documentação HPA (K8s)](https://kubernetes.io/docs/tasks/run-application/horizontal-pod-autoscale/)
* [Comando `kubectl autoscale`](https://kubernetes.io/docs/reference/kubectl/generated/kubectl_autoscale/)

---

## Instalação do Metrics Server

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

### Validação

```bash
kubectl top nodes
kubectl top pods
# Se aparecerem as métricas, está funcionando corretamente
```

### Configuração recomendada

```bash
kubectl edit deployment metrics-server -n kube-system
```

Adicione em `args`:

```yaml
args:
- --kubelet-insecure-tls
- --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
```

---

## Manifesto do HPA

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: monitor-app-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ .Values.deployment.name }}   # Nome do deployment (Helm)
  minReplicas: 2
  maxReplicas: 5
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50        # Escala com uso acima de 50% da CPU
    - type: Resource
      resource:
        name: memory
        target:
          type: Utilization
          averageUtilization: 70        # Escala com uso acima de 70% da Memória
```

---

## Gerar HPA via comando

```bash
kubectl autoscale deployment app-name --min=2 --max=10 --cpu-percent=70 --dry-run=client -o yaml > hpa.yaml
```

---

## Monitoramento do HPA

```bash
kubectl get hpa -w    # Exibe o status em tempo real
```

---

## Recurso obrigatório no Deployment

O HPA só funciona se o Deployment definir corretamente os recursos (`resources:`).
**Indentação errada = HPA não funciona (mostra "unknown")**

```yaml
spec:
  containers:
    - name: app
      image: {{ .Values.image }}
      ports:
        - containerPort: 5000
      resources:
        requests:
          cpu: "100m"
          memory: "64Mi"
        limits:
          cpu: "200m"      # Normalmente nao se limita cpu do container
          memory: "128Mi"
```