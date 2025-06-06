# KEDA – Kubernetes Event-Driven Autoscaling

## Referência

* [Documentação Oficial do KEDA](https://keda.sh/docs/1.5/concepts/scaling-deployments/)

---

## Instalação via Helm

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace
```

### Validação

```bash
kubectl get pods -n keda         # Verifica se o KEDA foi instalado
kubectl get scaledobjects        # Lista objetos de escala aplicados
```

---

## Exemplo de ScaledObject por uso de CPU

```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: cpu-scaledobject
spec:
  scaleTargetRef:
    name: {{ .Values.deployment.name }}   # Nome do deployment alvo (valor Helm)
  minReplicaCount: 1
  maxReplicaCount: 5                      # Sempre defina os limites
  triggers:
    - type: cpu
      metadata:
        type: Utilization
        value: "50"                       # Escala se uso da CPU > 50%
```

---

## Trigger para AWS SQS

```yaml
triggers:
  - type: aws-sqs-queue
    metadata:
      queueURLFromEnv: QUEUE_URL     # OU use diretamente 'queueURL'
      queueLength: "5"               # Tamanho da fila para escalar (default: 5)
      awsRegion: "us-east-1"         # Região da fila
      awsEndpoint: ""                # Opcional: endpoint customizado
```