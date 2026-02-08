### Criando um ServiceMonitor

Bem, chegou a hora de criarmos o nosso primeiro ServiceMonitor, mas antes precisamos ter uma aplicação para monitorarmos. Para isso, vamos criar uma aplicação com Nginx e utilizar o exporter do Nginx para monitorarmos o nosso serviço.
Vamos criar ainda um outro pod para que possamos criar um teste de carga para a nossa aplicação, realizando assim uma carga de até 1000 requisições por segundo.

Antes de mais nada, precisamos criar um ConfigMap onde terá a configuração que queremos para o nosso Nginx. Nada demais, somente a criação do nosso ConfigMap com o arquivo de configuração do Nginx, onde vamos definir a rota /nginx_status para expor as métricas do Nginx, além de expor a rota /metrics para expor as métricas do Nginx Exporter.

Vamos criar o nosso ConfigMap com o seguinte arquivo YAML:

```
apiVersion: v1 # versão da API
kind: ConfigMap # tipo de recurso, no caso, um ConfigMap
metadata: # metadados do recurso
  name: nginx-config # nome do recurso
data: # dados do recurso
  nginx.conf: | # inicio da definição do arquivo de configuração do Nginx
    server {
      listen 80;
      location / {
        root /usr/share/nginx/html;
        index index.html index.htm;
      }
      location /metrics {
        stub_status on;
        access_log off;
      }
    }
```
Agora vamos entender o que está acontecendo no nosso arquivo YAML.

- apiVersion: Versão da API do Kubernetes que estamos utilizando.
- kind: Tipo de objeto que estamos criando.
- metadata: Informações sobre o objeto que estamos criando.
- metadata.name: Nome do nosso objeto.
- data: Dados que serão utilizados no nosso ConfigMap.
- data.nginx.conf: A configuração do Nginx.
- Vamos criar o nosso ConfigMap com o seguinte comando:

```
kubectl apply -f nginx-config.yaml
```

Após o nosso ConfigMap ser criado, vamos verificar se o nosso 
ConfigMap está rodando:

```
kubectl get configmaps
```

Para criar a nossa aplicação, vamos utilizar o seguinte arquivo YAML:

```
apiVersion: apps/v1 # versão da API
kind: Deployment # tipo de recurso, no caso, um Deployment
metadata: # metadados do recurso 
  name: nginx-server # nome do recurso
spec: # especificação do recurso
  selector: # seletor para identificar os pods que serão gerenciados pelo deployment
    matchLabels: # labels que identificam os pods que serão gerenciados pelo deployment
      app: nginx # label que identifica o app que será gerenciado pelo deployment
  replicas: 3 # quantidade de réplicas do deployment
  template: # template do deployment
    metadata: # metadados do template
      labels: # labels do template
        app: nginx # label que identifica o app
      annotations: # annotations do template
        prometheus.io/scrape: 'true' # habilita o scraping do Prometheus
        prometheus.io/port: '9113' # porta do target
    spec: # especificação do template
      containers: # containers do template 
        - name: nginx # nome do container
          image: nginx # imagem do container do Nginx
          ports: # portas do container
            - containerPort: 80 # porta do container
              name: http # nome da porta
          volumeMounts: # volumes que serão montados no container
            - name: nginx-config # nome do volume
              mountPath: /etc/nginx/conf.d/default.conf # caminho de montagem do volume
              subPath: nginx.conf # subpath do volume
        - name: nginx-exporter # nome do container que será o exporter
          image: 'nginx/nginx-prometheus-exporter:0.11.0' # imagem do container do exporter
          args: # argumentos do container
            - '-nginx.scrape-uri=http://localhost/metrics' # argumento para definir a URI de scraping
          resources: # recursos do container
            limits: # limites de recursos
              memory: 128Mi # limite de memória
              cpu: 0.3 # limite de CPU
          ports: # portas do container
            - containerPort: 9113 # porta do container que será exposta
              name: metrics # nome da porta
      volumes: # volumes do template
        - configMap: # configmap do volume, nós iremos criar esse volume através de um configmap
            defaultMode: 420 # modo padrão do volume
            name: nginx-config # nome do configmap
          name: nginx-config # nome do volume
```

Agora vamos entender o que está acontecendo no nosso arquivo YAML.

- apiVersion: Versão da API do Kubernetes que estamos utilizando.
- kind: Tipo de objeto que estamos criando.
- metadata: Informações sobre o objeto que estamos criando.
- metadata.name: Nome do nosso objeto.
- spec: Especificações do nosso objeto.
- spec.selector: Selector que o ServiceMonitor irá utilizar para - encontrar os serviços que ele irá monitorar.
- spec.selector.matchLabels: Labels que o ServiceMonitor irá utilizar - para encontrar os serviços que ele irá monitorar.
- spec.selector.matchLabels.app: Label que o ServiceMonitor irá - utilizar para encontrar os serviços que ele irá monitorar.
- spec.replicas: Quantidade de réplicas que o nosso Deployment irá - criar.
- spec.template: Template que o nosso Deployment irá utilizar para - criar os pods.
- spec.template.metadata: Informações sobre o nosso pod.
- spec.template.metadata.labels: Labels que serão adicionadas ao nosso - pod.
- spec.template.metadata.labels.app: Label que será adicionada ao nosso - pod.
- spec.template.metadata.annotations: Annotations que serão adicionadas - ao nosso pod.
- spec.template.metadata.annotations.prometheus.io/scrape: Annotation - que será adicionada ao nosso pod.
- spec.template.metadata.annotations.prometheus.io/port: Annotation que - será adicionada ao nosso pod.
- spec.template.spec: Especificações do nosso pod.
- spec.template.spec.containers: Containers que serão criados no nosso - pod.
- spec.template.spec.containers.name: Nome do nosso container.
- spec.template.spec.containers.image: Imagem que será utilizada no - nosso container.
- spec.template.spec.containers.ports: Portas que serão expostas no - nosso container.
- spec.template.spec.containers.ports.containerPort: Porta que será - exposta no nosso container.
- spec.template.spec.containers.volumeMounts: Volumes que serão - montados no nosso container.
- spec.template.spec.containers.volumeMounts.name: Nome do volume que - será montado no nosso container.
- spec.template.spec.containers.volumeMounts.mountPath: Caminho que o - volume será montado no nosso container.
- spec.template.spec.containers.volumeMounts.subPath.nginx.conf: - Subpath que o volume será montado no nosso container.
- spec.template.spec.volumes: Volumes que serão criados no nosso pod.
- spec.template.spec.volumes.configMap: ConfigMap que será utilizado no - nosso volume.
- spec.template.spec.volumes.configMap.defaultMode: Modo de permissão - que o volume será criado.
- spec.template.spec.volumes.configMap.name: Nome do ConfigMap que será - utilizado no nosso volume.
- spec.template.spec.volumes.name: Nome do nosso volume.
Agora que já sabemos o que está acontecendo no nosso arquivo YAML, vamos criar o nosso Deployment com o seguinte comando:

```
kubectl apply -f nginx-deployment.yaml
```

Após o nosso Deployment ser criado, vamos verificar se o nosso pod está rodando:

```
kubectl get pods
```

Podemos ver o deployment que acabamos de criar através do comando:

```
kubectl get deployments
```

Agora o que precisamos é criar um Service para expor o nosso deployment. Vamos criar o nosso Service com o seguinte arquivo YAML:

```
apiVersion: v1 # versão da API
kind: Service # tipo de recurso, no caso, um Service
metadata: # metadados do recurso
  name: nginx-svc # nome do recurso
  labels: # labels do recurso
    app: nginx # label para identificar o svc
spec: # especificação do recurso
  ports: # definição da porta do svc 
  - port: 9113 # porta do svc
    name: metrics # nome da porta
  selector: # seletor para identificar os pods/deployment que esse svc irá expor
    app: nginx # label que identifica o pod/deployment que será exposto
```

Agora vamos entender o que está acontecendo no nosso arquivo YAML.

- apiVersion: Versão da API do Kubernetes que estamos utilizando.
- kind: Tipo de objeto que estamos criando.
- metadata: Informações sobre o objeto que estamos criando.
- metadata.name: Nome do nosso objeto.
- spec: Especificações do nosso objeto.
- spec.selector: Selector que o Service irá utilizar para encontrar os - pods que ele irá expor.
- spec.selector.app: Label que o Service irá utilizar para encontrar os - pods que ele irá expor.
- spec.ports: Configurações das portas que serão expostas no nosso - Service.
- spec.ports.protocol: Protocolo que será utilizado na porta que será - exposta.
- spec.ports.port: Porta que será exposta no nosso Service.
- spec.ports.name: Nome da porta que será exposta no nosso Service.
Vamos criar o Service com o seguinte comando:

```
kubectl apply -f nginx-service.yaml
```

Após o nosso Service ser criado, vamos verificar se o nosso Service está rodando:

```
kubectl get services
```

Pronto, tudo criado!

Acho que já temos tudo criado, agora vamos verificar se o nosso Nginx está rodando e se as métricas estão sendo expostas.

Vamos verificar se o nosso Nginx está rodando com o seguinte comando:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80
```

Vamos verificar se as métricas do Nginx estão sendo expostas com o seguinte comando:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80/nginx_status
```

Vamos verificar se as métricas do Nginx Exporter estão sendo expostas com o seguinte comando:

```
curl http://<EXTERNAL-IP-DO-SERVICE>:80/metrics
```
Ótimo, agora você já sabe como que faz para criar um Service no Kubernetes e expor as métricas do Nginx e do Nginx Exporter. 😄

