# Argo CD

## Referências

* [Argo CD Docs](https://argo-cd.readthedocs.io/en/stable/)
* [Argo CD CLI](https://argo-cd.readthedocs.io/en/stable/user-guide/commands/argocd/)

---

## Helm Install

```bash
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update
```

```bash
# Criação do namespace
kubectl create namespace argocd
```

```bash
# Instalação do Argo CD via Helm
helm upgrade --install argocd argo/argo-cd \
  --namespace argocd
```

---

## Validação da Instalação

```bash
# Verificar pods e serviços
kubectl get pods -n argocd
kubectl get svc -n argocd
```

---

## Acesso ao Argo CD

### Opção 1: Port-Forward (mais simples, ambiente de lab)

```bash
kubectl port-forward svc/argocd-server -n argocd 8080:443
```

Acesse no navegador:
👉 [https://localhost:8080](https://localhost:8080)

---

### Opção 2: LoadBalancer (produção/teste público)

```bash
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
kubectl get svc argocd-server -n argocd
```

---

## Senha Inicial

```bash
# Pegar senha inicial do usuário admin
kubectl get secret argocd-initial-admin-secret -n argocd \
  -o jsonpath="{.data.password}" | base64 -d
```

---

## Instalação do ArgoCD CLI

### Linux

```bash
wget https://github.com/argoproj/argo-cd/releases/latest/download/argocd-linux-amd64 -O argocd
chmod +x argocd
sudo mv argocd /usr/local/bin/
```

### Mac

```bash
brew install argocd
```

---

## Login via CLI

### Se estiver usando **port-forward**

```bash
ARGOCD_PWD=$(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)

argocd login localhost:8080 \
  --username admin \
  --password $ARGOCD_PWD \
  --insecure
```

### Se estiver usando **LoadBalancer**

```bash
ARGOCD_SERVER=<EXTERNAL-IP-do-argocd-server>
ARGOCD_PWD=$(kubectl get secret argocd-initial-admin-secret -n argocd -o jsonpath="{.data.password}" | base64 -d)

argocd login $ARGOCD_SERVER \
  --username admin \
  --password $ARGOCD_PWD \
  --insecure
```

---

## Criar Aplicação de Exemplo

```bash
argocd app create guestbook \
  --repo https://github.com/argoproj/argocd-example-apps.git \
  --path guestbook \
  --dest-server https://kubernetes.default.svc \
  --dest-namespace default
```

```bash
# Sincronizar aplicação
argocd app sync guestbook
```

```bash
# Verificar status
argocd app list
argocd app get guestbook
```

---

## Validação

```bash
# Verificar recursos no cluster
kubectl get pods,svc -n default

# Verificar apps no ArgoCD
argocd app list
```

