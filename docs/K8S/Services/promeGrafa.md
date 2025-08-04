# Setup Prometheus + Grafana no k8s

## Referências

* [AWS Docs – Deploy Prometheus no EKS](https://docs.aws.amazon.com/eks/latest/userguide/deploy-prometheus.html)
* [Artigo Medium – Prometheus + Grafana no K8s](https://medium.com/@akilblanchard09/monitoring-a-kubernetes-cluster-using-prometheus-and-grafana-8e0f21805ea9)

---

## Instalação via Helm

### Adicionar repositórios

```bash
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update
```

### Instalar Prometheus

```bash
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring --create-namespace
```

### Instalar Grafana

```bash
helm install grafana grafana/grafana \
  --namespace monitoring --create-namespace
```

---

## Validação

```bash
kubectl get all -n monitoring
```

---

## Acesso via port-forward

### Grafana

```bash
kubectl port-forward svc/grafana 3000:80 -n monitoring
```

Acesse: [http://localhost:3000](http://localhost:3000)
Usuário: `admin`
Senha:

```bash
kubectl get secret grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode
```

---

### Prometheus

```bash
kubectl port-forward svc/prometheus-server 9090:80 -n monitoring
```

Acesse: [http://localhost:9090](http://localhost:9090)

---

## Instalação do Prometheus e Grafana com Node Selector

### Criar arquivos de configuração

#### prometheus-values.yaml
```yaml
nodeSelector:
  nodegroup: monitoring

prometheus:
  nodeSelector:
    nodegroup: monitoring
  service:
    type: ClusterIP

alertmanager:
  nodeSelector:
    nodegroup: monitoring

grafana:
  nodeSelector:
    nodegroup: monitoring
```

### grafana-values.yaml
```yaml
nodeSelector:
  nodegroup: monitoring

grafana:
  additionalDataSources:
    - name: Prometheus
      type: prometheus
      url: http://kube-prometheus-kube-prome-prometheus.monitoring.svc.cluster.local:9090
      access: proxy
      isDefault: true
```

## Comandos de instalação

```bash
# Adicionar repositórios Helm
helm repo add prometheus-community https://prometheus-community.github.io/helm-charts
helm repo add grafana https://grafana.github.io/helm-charts
helm repo update

# Instalar Prometheus Stack
helm install kube-prometheus prometheus-community/kube-prometheus-stack \
  -f prometheus-values.yaml \
  --namespace monitoring \
  --create-namespace

# Instalar Grafana  
helm install grafana grafana/grafana \
  -f grafana-values.yaml \
  --namespace monitoring \
  --create-namespace
```

## Verificar instalação
```bash
kubectl get pods -n monitoring -o wide
```

## Acessar os serviços

```bash
# Prometheus
kubectl port-forward -n monitoring pod/prometheus-kube-prometheus-kube-prome-prometheus-0 9090

# Grafana
kubectl port-forward svc/grafana 3000:80 -n monitoring
```
**Credenciais Grafana:**
- Usuário: admin
- Senha: `kubectl get secret grafana -n monitoring -o jsonpath="{.data.admin-password}" | base64 --decode`  