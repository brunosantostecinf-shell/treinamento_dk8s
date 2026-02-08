Startup Probe

Chegou a hora de falar sobre a probe, que na minha humilde opinião, é a menos utilizada, mas que é muito importante, a startupProbe.

Ela é a responsável por verificar se o nosso container foi inicializado corretamente, e se ele está pronto para receber requisições.

Ele é muito parecido com a readinessProbe, mas a diferença é que a startupProbe é executada apenas uma vez no começo da vida do nosso container, e a readinessProbe é executada de tempos em tempos.

Para entender melhor, vamos ver um exemplo criando um arquivo chamado nginx-startup.yaml:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-deployment
  strategy: {}
  template:
    metadata:
      labels:
        app: nginx-deployment
    spec:
      containers:
      - image: nginx:1.19.2
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
        startupProbe: # Onde definimos a nossa probe de inicialização
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
```

Agora vamos aplicar a nossa configuração:

```
kubectl apply -f nginx-startup.yaml
``` 

Quando você tentar aplicar, receberá um erro, pois a successThreshold não pode ser maior que 1, pois a startupProbe é executada apenas uma vez, lembra?

Da mesma forma o failureThreshold não pode ser maior que 1, então vamos alterar o nosso arquivo para:

```
...
        startupProbe: # Onde definimos a nossa probe de inicialização
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
``` 

Agora vamos aplicar novamente:

```
kubectl apply -f nginx-startup.yaml
``` 

Pronto, aplicado! Ufa! \o/

Perceba que sua definição é super parecida com a readinessProbe, mas lembre-se, ela somente será executada uma vez, quando o container for inicializado. Portanto, se alguma coisa acontecer de errado depois disso, ele não irá te salvar, pois ele não irá executar novamente.

Por isso é super importante sempre ter uma combinação entre as probes, para que você tenha um container mais resiliente e que problemas possam ser detectados mais rapidamente.

Vamos ver se os nossos Pods estão saudáveis:

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-6fbd5f9794-66sww   1/1     Running   0          2m12s
nginx-deployment-6fbd5f9794-cmwq8   1/1     Running   0          2m12s
nginx-deployment-6fbd5f9794-kvrp8   1/1     Running   0          2m12s
``` 

Caso você queira conferir se a nossa probe está lá, basta usar o comando:

```
kubectl describe pod nginx-deployment-6fbd5f9794-66sww
``` 

E você verá algo parecido com isso:

```
    Startup:      http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=1
``` 


