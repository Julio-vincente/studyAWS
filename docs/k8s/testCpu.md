# Stress de CPU no Kubernetes

## Referências

* [Artigo Medium](https://medium.com/@email2smohanty/stress-testing-kubernetes-k8s-and-openshift-nodes-using-stress-ng-35b6a9752c8d)
* [Guia stress-ng](https://blog.ironlinux.com.br/estressando-mem-disco-e-cpu-com-stress-ng-debian9/)
* [Imagem Docker polinux/stress-ng](https://hub.docker.com/r/polinux/stress)

---

## Namespace para o teste

```bash
kubectl create namespace monitor
```

---

## Manifesto de teste (CPU Stress)

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Values.test.name }}        # Nome vindo do values.yaml (Helm)
  namespace: monitor
spec:
  containers:
    - name: stress
      image: polinux/stress-ng
      command: [ "stress-ng" ]
      args: [ "--cpu", "2", "--cpu-method", "fft", "--timeout", "300s" ]  
      # Usa 2 CPUs com carga alta por 300 segundos (FFT)
  restartPolicy: Never
```

---

## Aplicar o teste

```bash
kubectl apply -f test-memory-cpu.yaml
```

### Validar execução

```bash
kubectl get pods -n monitor
```