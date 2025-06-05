# Stress CPU workload

## Docs

HELP[](https://medium.com/@email2smohanty/stress-testing-kubernetes-k8s-and-openshift-nodes-using-stress-ng-35b6a9752c8d).
Stress-ng Docs:[](https://blog.ironlinux.com.br/estressando-mem-disco-e-cpu-com-stress-ng-debian9/).
Poli Linux/Container:[](https://hub.docker.com/r/polinux/stress).

## Workspace

```bash
k create namespace monitor
```

---

## Test Manifest
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: {{ .Values.test.name }} # valor do values do helm 
  namespace: monitor
spec:
  containers:
    - name: stress
      image: polinux/stress-ng # utilizando essa imagem qualquer duvida olhar docs
      command: [ 'stress-ng' ]
      args: [ "--cpu", "2", "--cpu-method", "fft", "--timeout", "300s" ] # fazendo com que use 2 cpus estrasando ela durante 300s
  restartPolicy: Never
```

Aplicar:
```bash
k apply -f test-memory-cpu.yaml
```

Validation:
```bash
k get pods -n monitor
```
