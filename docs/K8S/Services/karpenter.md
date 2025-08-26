# Karpenter

## Referências

* [Karpenter Docs](https://karpenter.sh/docs/getting-started/)
* [AWS EKS + Karpenter](https://karpenter.sh/docs/getting-started/getting-started-with-eksctl/)

---

## Helm Install

```bash
helm repo add karpenter https://charts.karpenter.sh
helm repo update
```

```bash
# Namespace e ServiceAccount para o controller
kubectl create namespace karpenter
kubectl annotate serviceaccount -n karpenter default \
  eks.amazonaws.com/role-arn=arn:aws:iam::<ACCOUNT_ID>:role/karpenter-controller-role \
  --overwrite
```

```bash
# Instalação do Karpenter
helm upgrade --install karpenter karpenter/karpenter \
  --namespace karpenter \
  --set settings.clusterName=<CLUSTER_NAME> \
  --set settings.clusterEndpoint=<CLUSTER_ENDPOINT> \
  --set settings.aws.defaultInstanceProfile=karpenter-node-instance-profile \
  --set serviceAccount.annotations."eks\.amazonaws\.com/role-arn"=arn:aws:iam::<ACCOUNT_ID>:role/karpenter-controller-role
```

---

## NodeClass e NodePool

```yaml
# ec2nodeclass.yaml
apiVersion: karpenter.k8s.aws/v1beta1
kind: EC2NodeClass
metadata:
  name: default-ec2
spec:
  amiFamily: AL2023
  role: karpenter-node-role
  subnetSelectorTerms:
    - tags:
        kubernetes.io/cluster/<CLUSTER_NAME>: shared
  securityGroupSelectorTerms:
    - tags:
        kubernetes.io/cluster/<CLUSTER_NAME>: owned
```

```yaml
# nodepool.yaml
apiVersion: karpenter.sh/v1beta1
kind: NodePool
metadata:
  name: default
spec:
  template:
    spec:
      nodeClassRef:
        name: default-ec2
      requirements:
        - key: "node.kubernetes.io/instance-type"
          operator: In
          values: ["t3.medium","t3.large"]
  limits:
    cpu: "1000"
    memory: 1000Gi
```

```bash
kubectl apply -f ec2nodeclass.yaml
kubectl apply -f nodepool.yaml
```

---

## Validação

```bash
# Verificar pods do Karpenter
kubectl get pods -n karpenter

# Teste de escala (um deployment que pede 5 réplicas)
kubectl create deployment inflate --image=public.ecr.aws/eks-distro/kubernetes/pause:3.2 --replicas=5
kubectl get nodes -o wide
```

