Um dos principais recursos que o Kube-Prometheus utiliza é o ServiceMonitor. O ServiceMonitor é um recurso do Prometheus Operator que permite que você configure o Prometheus para monitorar um serviço. Para isso, você precisa criar um ServiceMonitor para cada serviço que você deseja monitorar.

O Kube-Prometheus já vem com vários ServiceMonitors configurados, como por exemplo o ServiceMonitor do API Server, do Node Exporter, do Blackbox Exporter, etc.

```bash
kubectl get servicemonitors -n monitoring
NAME                      AGE
alertmanager              17m
blackbox-exporter         17m
coredns                   17m
grafana                   17m
kube-apiserver            17m
kube-controller-manager   17m
kube-scheduler            17m
kube-state-metrics        17m
kubelet                   17m
node-exporter             17m
prometheus-adapter        17m
prometheus-k8s            17m
prometheus-operator       17m
```

Para ver o conteúdo de um ServiceMonitor, basta executar o seguinte comando:

```bash
kubectl get servicemonitor prometheus-k8s -n monitoring -o yaml
```
Nesse caso estamos pegando o ServiceMonitor do Prometheus, mas você pode pegar o ServiceMonitor de qualquer outro serviço.

```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"monitoring.coreos.com/v1","kind":"ServiceMonitor","metadata":{"annotations":{},"labels":{"app.kubernetes.io/component":"prometheus","app.kubernetes.io/instance":"k8s","app.kubernetes.io/name":"prometheus","app.kubernetes.io/part-of":"kube-prometheus","app.kubernetes.io/version":"2.41.0"},"name":"prometheus-k8s","namespace":"monitoring"},"spec":{"endpoints":[{"interval":"30s","port":"web"},{"interval":"30s","port":"reloader-web"}],"selector":{"matchLabels":{"app.kubernetes.io/component":"prometheus","app.kubernetes.io/instance":"k8s","app.kubernetes.io/name":"prometheus","app.kubernetes.io/part-of":"kube-prometheus"}}}}
  creationTimestamp: "2023-01-23T19:08:26Z"
  generation: 1
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.41.0
  name: prometheus-k8s
  namespace: monitoring
  resourceVersion: "4100"
  uid: 6042e08c-cf18-4622-9860-3ff43e696f7c
spec:
  endpoints:
  - interval: 30s
    port: web
  - interval: 30s
    port: reloader-web
  selector:
    matchLabels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
```

Eu vou dar uma limpada nessa saída para ficar mais fácil de entender:

```bash
apiVersion: monitoring.coreos.com/v1
kind: ServiceMonitor
metadata:
  annotations:
  labels:
    app.kubernetes.io/component: prometheus
    app.kubernetes.io/instance: k8s
    app.kubernetes.io/name: prometheus
    app.kubernetes.io/part-of: kube-prometheus
    app.kubernetes.io/version: 2.41.0
  name: prometheus-k8s
  namespace: monitoring
spec:
  endpoints:
  - interval: 30s
    port: web
  - interval: 30s
    port: reloader-web
  selector:
    matchLabels:
      app.kubernetes.io/component: prometheus
      app.kubernetes.io/instance: k8s
      app.kubernetes.io/name: prometheus
      app.kubernetes.io/part-of: kube-prometheus
```
Pronto, eu tirei algumas informações que não são importantes para a criação do ServiceMonitor, elas apenas trazer as informações do service monitor que foi criado e que pegamos a saída.

Com o arquivo limpo, podemos entender melhor o que está acontecendo.

- apiVersion: Versão da API do Kubernetes que estamos utilizando.
- kind: Tipo de objeto que estamos criando.
- metadata: Informações sobre o objeto que estamos criando.
- metadata.annotations: Anotações que podemos adicionar ao nosso objeto.
- metadata.labels: Labels que podemos adicionar ao nosso objeto.
- metadata.name: Nome do nosso objeto.
- metadata.namespace: Namespace onde o nosso objeto será criado.
- spec: Especificações do nosso objeto.
- spec.endpoints: Endpoints que o nosso ServiceMonitor irá monitorar.
- spec.endpoints.interval: Intervalo de tempo que o Prometheus irá - fazer a coleta de métricas.
- spec.endpoints.port: Porta que o Prometheus irá utilizar para coletar - as métricas.
- spec.selector: Selector que o ServiceMonitor irá utilizar para encontrar os serviços que ele irá monitorar.

Com isso, sabemos que o ServiceMonitor do Prometheus irá monitorar os serviços que possuem as labels app.kubernetes.io/component: prometheus, app.kubernetes.io/instance: k8s, app.kubernetes.io/name: prometheus e app.kubernetes.io/part-of: kube-prometheus, e que ele irá monitorar as portas web e reloader-web com um intervalo de 30 segundos. É fácil ou não é?

Então sempre que precisarmos criar um ServiceMonitor para monitorar algum serviço, basta criarmos um arquivo YAML com as informações que precisamos e aplicarmos em nosso cluster.

