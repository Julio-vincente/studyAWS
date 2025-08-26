# Nginx Ingress

## Referência

* [Documentação Oficial do NGINX Ingress](https://kubernetes.github.io/ingress-nginx/)

---

## Instalação via Helm do NGINX Ingress Controller

```bash
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace \
  --set controller.replicaCount=2 \
  --set controller.service.type=LoadBalancer
```

---

## Exemplo de Canary Deployment separando 25% para a versão canary

### Ingress Stable (75%)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-api-ingress
  namespace: default # namespace default mudar para o seu
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # O tráfego principal que será 75% vai para o stable
            name: stable-service
            port:
              number: 80
```

---

### Ingress Canary (25%)

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-api-ingress-canary
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/canary: "true"          # ativa o modo canary
    nginx.ingress.kubernetes.io/canary-weight: "25"     # separa 25% para o canary
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # o tráfego do canary será 25%
            name: canary-service
            port:
              number: 80
```

---

