### Adicione um Taint "dedicated=database:NoSchedule" no node "worker-02" e depois remova-o.

```bash
kubectl taint node worker-02 dedicated=database:NoSchedule
kubectl taint node worker-02 dedicated=database:NoSchedule-
```

### Configure um Pod para rodar apenas em nodes com a label "disktype=ssd" usando requiredDuringSchedulingIgnoredDuringExecution.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: ssd-pod
spec:
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: disktype
            operator: In
            values:
            - ssd
  containers:
  - name: app
    image: nginx:1.26
```

### Pod com Toleration
Crie um Pod que tolere o Taint "dedicated=database:NoSchedule" para poder rodar no node "worker-02".

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: db-pod
spec:
  tolerations:
  - key: "dedicated"
    operator: "Equal"
    value: "database"
    effect: "NoSchedule"
  containers:
  - name: db
    image: redis:7
```

### Pod AntiAffinity para alta disponibilidade
Configure um Deployment com podAntiAffinity para garantir que os Pods nunca fiquem no mesmo node, usando requiredDuringSchedulingIgnoredDuringExecution com topologyKey "kubernetes.io/hostname".

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: ha-app
spec:
  replicas: 3
  selector:
    matchLabels:
      app: ha-web
  template:
    metadata:
      labels:
        app: ha-web
    spec:
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchLabels:
                  app: ha-web
              topologyKey: kubernetes.io/hostname
      containers:
        - name: web
          image: nginx:1.26
```