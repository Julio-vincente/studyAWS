# Setup Prometheus + Grafana no Kubernetes

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
  --namespace monitoring
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

Claro! Aqui está exatamente o que você pediu: os comandos **helm** com o parâmetro para **indicar a label do Node Group (role=monitoring) no nodeSelector** — direto no comando Helm.

---

# Helm commands para instalar Prometheus e Grafana no Node Group com label que no caso sera `role=monitoring`

### Prometheus

```bash
helm install prometheus prometheus-community/prometheus \
  --namespace monitoring --create-namespace \
  --set server.nodeSelector.role=monitoring \
  --set alertmanager.nodeSelector.role=monitoring \
  --set pushgateway.nodeSelector.role=monitoring \
  --set kubeStateMetrics.nodeSelector.role=monitoring
```

### Grafana

```bash
helm install grafana grafana/grafana \
  --namespace monitoring \
  --set nodeSelector.role=monitoring
```

---
