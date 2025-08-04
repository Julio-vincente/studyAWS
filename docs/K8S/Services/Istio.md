# Istio

## Referências

* [Istio install]( https://istio.io/latest/docs/setup/install/helm/ )
* [Istioctl cmd]( https://istio.io/latest/docs/reference/commands/istioctl/ )

---

## Helm Install

```bash
helm repo add istio https://istio-release.storage.googleapis.com/charts
helm repo update
```

```bash
# Istio Base install
helm upgrade --install istio-base istio/base -n istio-system --create-namespace
# Istio istiod install
helm upgrade --install istiod istio/istiod -n istio-system
# Istio ingress install
helm upgrade --install istio-ingress istio/gateway -n istio-ingress --create-namespace

kubectl label namespace istio-ingress istio-injection=enabled
```

## Validação

```bash
helm ls -n istio-system
helm ls -n istio-ingress
```