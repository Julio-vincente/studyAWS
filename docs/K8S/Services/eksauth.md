# EKS – ClusterRole e ClusterRoleBinding para Console

## Referências

* [AWS EKS – Cluster Authentication](https://docs.aws.amazon.com/eks/latest/userguide/cluster-auth.html)
* [RBAC Kubernetes](https://kubernetes.io/docs/reference/access-authn-authz/rbac/)
* [eksctl Docs](https://eksctl.io/usage/creating-and-managing-clusters/)

---

## Criando com **kubectl**

### ClusterRole

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: eks-console-dashboard-full-access
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "endpoints", "namespaces", "events"]
  verbs: ["get", "list", "watch"]
```

```bash
kubectl apply -f clusterrole-eks-console.yaml
```

### ClusterRoleBinding

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: eks-console-dashboard-full-access-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: eks-console-dashboard-full-access
subjects:
- kind: User
  name: eks-console-dashboard-reporter
  apiGroup: rbac.authorization.k8s.io
```

```bash
kubectl apply -f clusterrolebinding-eks-console.yaml
```

---

## Criando com **eksctl**

### Arquivo de Configuração (`eks-console-access.yaml`)

```yaml
apiVersion: eksctl.io/v1alpha5
kind: ClusterConfig

metadata:
  name: jam-cluster
  region: us-east-1   # ajuste para a sua região

kubernetes:
  clusterRoles:
    - name: eks-console-dashboard-full-access
      rules:
        - apiGroups: [""]
          resources: ["nodes", "pods", "services", "endpoints", "namespaces", "events"]
          verbs: ["get", "list", "watch"]

  clusterRoleBindings:
    - metadata:
        name: eks-console-dashboard-full-access-binding
      roleRef:
        apiGroup: rbac.authorization.k8s.io
        kind: ClusterRole
        name: eks-console-dashboard-full-access
      subjects:
        - kind: User
          name: eks-console-dashboard-reporter
          apiGroup: rbac.authorization.k8s.io
```

### Aplicar

```bash
eksctl utils apply -f eks-console-access.yaml --approve
```

---

## Validação

### Pelo Console AWS

1. Vá em **EKS > Clusters > jam-cluster > Nodes**
   → Os nodes agora devem aparecer.

2. Vá em **Events** e procure pelo evento com **Reason = RegisteredNode**.

   Possíveis valores no campo **From**:

   * `kubelet` (mais comum, o agente no node)
   * `node-controller` (controlador do kube-controller-manager)
   * `cloud-controller-manager` (casos raros em ambientes cloud)

### Pelo kubectl

```bash
kubectl get events -A --field-selector reason=RegisteredNode
```
