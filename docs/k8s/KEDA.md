# KEDA  

How to setup KEDA in K8s Cluster

## DOCS

[KEDA Docs](https://keda.sh/docs/1.5/concepts/scaling-deployments/).

## Workspace

### Installation

```bash
helm repo add kedacore https://kedacore.github.io/charts
helm repo update
helm install keda kedacore/keda --namespace keda --create-namespace```
```

Validation:
```bash
kubectl get pods -n keda # valida depois da instalacao do servico 
kubectl get scaledobjects # lista depois do deploy
```

---

## KEDA Manifest
```yaml
apiVersion: keda.sh/v1alpha1
kind: ScaledObject
metadata:
  name: cpu-scaledobject
spec:
  scaleTargetRef:
    name: {{ .Values.deployment.name }} # valor do helm
  minReplicaCount: 1
  maxReplicaCount: 5 
  # sempre definir o min e o max de replicas ( OBRIGATORIO )
  triggers: 
  - type: cpu
    metadata:
      type: Utilization
      value: "50"
	# define o trigger para quando tiver mais que 50% da cpu sendo utilizada ira escalar
```

### AWS SQS/SNS
```yaml
triggers:
  - type: aws-sqs-queue
    metadata:
      queueURL: "url da fila"
      queueURLFromEnv: QUEUE_URL # Em vez de queueURL pode se usar esse
      queueLength: "5" # default 5
      awsRegion: "us-east-1" # regia definida
      awsEndpoint: "" # optional
```