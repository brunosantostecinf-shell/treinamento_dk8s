Vamos criar o nosso ServiceMonitor com o seguinte arquivo YAML:

```
apiVersion: monitoring.coreos.com/v1 # versão da API
kind: ServiceMonitor # tipo de recurso, no caso, um ServiceMonitor do Prometheus Operator
metadata: # metadados do recurso
  name: nginx-servicemonitor # nome do recurso
  labels: # labels do recurso
    app: nginx # label que identifica o app
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão monitorados
    matchLabels: # labels que identificam os pods que serão monitorados
      app: nginx # label que identifica o app que será monitorado
  endpoints: # endpoints que serão monitorados
    - interval: 10s # intervalo de tempo entre as requisições
      path: /metrics # caminho para a requisição
      targetPort: 9113 # porta do target
```

Agora vamos entender o que está acontecendo no nosso arquivo YAML.

- - apiVersion: Versão da API do Kubernetes que estamos utilizando.
- kind: Tipo de objeto que estamos criando, no nosso caso, um ServiceMonitor.
- metadata: Informações sobre o objeto que estamos criando.
- metadata.name: Nome do nosso objeto.
- metadata.labels: Labels que serão utilizadas para identificar o nosso objeto.
- spec: Especificações do nosso objeto.
- spec.selector: Seletor que será utilizado para identificar o nosso Service.
- spec.selector.matchLabels: Labels que serão utilizadas para identificar o nosso Service, no nosso caso, o Service que tem a label - app: nginx.
- spec.endpoints: Endpoints que serão monitorados pelo Prometheus.
- spec.endpoints.interval: Intervalo de tempo que o Prometheus irá capturar as métricas, no nosso caso, 15 segundos.
- spec.endpoints.path: Caminho que o Prometheus irá fazer a requisição para capturar as métricas, no nosso caso, /metrics.
- spec.endpoints.targetPort: Porta que o Prometheus irá fazer a requisição para capturar as métricas, no nosso caso, 9113.
- Vamos criar o nosso ServiceMonitor com o seguinte comando:

```
kubectl apply -f nginx-service-monitor.yaml
```

Após o nosso ServiceMonitor ser criado, vamos verificar se o nosso ServiceMonitor está rodando:

```
kubectl get servicemonitors
```

Maravilha! Agora que já temos o nosso Nginx rodando e as métricas sendo expostas, vamos verificar se o Prometheus está capturando as métricas do Nginx e do Nginx Exporter.

Vamos fazer o port-forward do Prometheus para acessar o Prometheus localmente:

```
kubectl port-forward -n monitoring svc/prometheus-k8s 39090:9090
```
E agora vamos usar o curl para verificar se o Prometheus está capturando as métricas do Nginx e do Nginx Exporter:

```
curl http://localhost:39090/api/v1/targets
```

Pronto, agora você já sabe como que faz para criar um Service no Kubernetes, expor as métricas do Nginx e do Nginx Exporter e ainda criar um ServiceMonitor para o seu Service ficar monitorado pelo Prometheus. \o/

É muito importante que você saiba que o Prometheus não captura as métricas automaticamente, ele precisa de um ServiceMonitor para capturar as métricas. 😄


