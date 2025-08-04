# Nginx Ingress

## Referência

* [Documentação Oficial do NGINX Ingress](https://kubernetes.github.io/ingress-nginx/)

---

## Instalação via Helm do NGINX Ingress Controller

```bash
helm upgrade --install ingress-nginx ingress-nginx \
  --repo https://kubernetes.github.io/ingress-nginx \
  --namespace ingress-nginx --create-namespace
```

---

## Exemplo de Canary Deployment seperando 25% para a versão canary

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-api-ingress
  namespace: default # namespace default mudar para o seu
  annotations:
    nginx.ingress.kubernetes.io/canary: "true" # ativa o modo canary
    nginx.ingress.kubernetes.io/canary-weight: "25" # essa annotation separa 25% para canary
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # O trafego principal que seria 75% vai ser utilizado para o stable          
            name: stable-service
            port:
              number: 80
              
---

# Precisamos de um segundo Ingress porque precisamos definiar o trafego do canary
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-api-ingress-canary
  namespace: default
  annotations:
    # essa annotation defini que o ingress vai receber o trafego do canary
    nginx.ingress.kubernetes.io/canary: "true" 
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            # o trafego do canary vai ser 25%
            name: canary-service
            port:
              number: 80
```

---