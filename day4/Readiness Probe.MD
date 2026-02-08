Readiness Probe

A readinessProbe é uma forma de o Kubernetes verificar se o seu container está pronto para receber tráfego, se ele está pronto para receber requisições vindas de fora.

Essa é a nossa probe de leitura, ela fica verificando se o nosso container está pronto para receber requisições, e se estiver pronto, ele irá receber requisições, caso contrário, ele não irá receber requisições, pois será removido do endpoint do serviço, fazendo com que o tráfego não chegue até ele.

Ainda iremos ver o que é service e endpoint, mas por enquanto, basta saber que o endpoint é o endereço que o nosso service irá usar para acessar o nosso Pod. Mas vamos ter um dia inteiro para falar sobre service e endpoint, então, relaxa.

 

Voltando ao assunto, a nossa probe da vez irá garantir que o nosso Podestá saudável para receber requisições.

Vamos para um exemplo para ficar mais claro.

Para o nosso exemplo, vamos criar um arquivo chamado nginx-readiness.yaml e vamos colocar o seguinte conteúdo:

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
        readinessProbe: # Onde definimos a nossa probe de leitura
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: / # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
``` 

Vamos ver se os nossos Pods estão rodando:

```
kubectl get pods
 

NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-fbdc9b65f-trnnz   0/1     Running   0          6s
nginx-deployment-fbdc9b65f-z8n4m   0/1     Running   0          6s
nginx-deployment-fbdc9b65f-zn8zh   0/1     Running   0          6s
``` 

Podemos ver que agora os Pods demoram um pouco mais para ficarem prontos, pois estamos executando a nossa readinessProbe, e por esse motivo temos que aguardar os 10 segundos inicias que definimos para que seja executada a primeira vez a nossa probe, lembra?

Se você aguardar um pouco, você verá que os Pods irão ficar prontos, e você pode ver isso executando o comando:

```
kubectl get pods
 

NAME                               READY   STATUS    RESTARTS   AGE
nginx-deployment-fbdc9b65f-trnnz   1/1     Running   0          30s
nginx-deployment-fbdc9b65f-z8n4m   1/1     Running   0          30s
nginx-deployment-fbdc9b65f-zn8zh   1/1     Running   0          30s
``` 

Pronto, como mágica agora os nossos Pods estão prontos para receber requisições.

Vamos dar uma olhada no describe do nosso Pod:

```
kubectl describe pod nginx-deployment-fbdc9b65f-trnnz
 

Name:             nginx-deployment-fbdc9b65f-trnnz
Namespace:        default
Priority:         0
Service Account:  default
Node:             ip-192-168-39-119.ec2.internal/192.168.39.119
Start Time:       Thu, 16 Mar 2023 19:10:07 +0100
Labels:           app=nginx-deployment
                  pod-template-hash=fbdc9b65f
Annotations:      kubernetes.io/psp: eks.privileged
Status:           Running
IP:               192.168.49.40
IPs:
  IP:           192.168.49.40
Controlled By:  ReplicaSet/nginx-deployment-fbdc9b65f
Containers:
  nginx:
    Container ID:   docker://09538e27e29c5c649efa88fe148336abd5a47dd4e5a8d32b40b268fb1818dfc4
    Image:          nginx:1.19.2
    Image ID:       docker-pullable://nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 16 Mar 2023 19:10:08 +0100
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Readiness:    http-get http://:80/ delay=10s timeout=5s period=10s #success=2 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-zpfvb (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-zpfvb:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   Burstable
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  60s   default-scheduler  Successfully assigned default/nginx-deployment-fbdc9b65f-trnnz to ip-192-168-39-119.ec2.internal
  Normal  Pulled     59s   kubelet            Container image "nginx:1.19.2" already present on machine
  Normal  Created    59s   kubelet            Created container nginx
  Normal  Started    59s   kubelet            Started container nginx
``` 

Pronto, a nossa probe está lá e funcionando, e com isso podemos garantir que os nossos Pods estão prontos para receber requisições.

Vamos mudar o nosso path para /giropops e ver o que acontece:

```
...
        readinessProbe: # Onde definimos a nossa probe de leitura
          httpGet: # O tipo de teste que iremos executar, neste caso, iremos executar um teste HTTP
            path: /giropops # O caminho que iremos testar
            port: 80 # A porta que iremos testar
          initialDelaySeconds: 10 # O tempo que iremos esperar para executar a primeira vez a probe
          periodSeconds: 10 # De quanto em quanto tempo iremos executar a probe
          timeoutSeconds: 5 # O tempo que iremos esperar para considerar que a probe falhou
          successThreshold: 2 # O número de vezes que a probe precisa passar para considerar que o container está pronto
          failureThreshold: 3 # O número de vezes que a probe precisa falhar para considerar que o container não está pronto
 

kubectl apply -f nginx-deployment.yaml
 

deployment.apps/nginx-deployment configured
``` 

Muito bom, agora vamos ver o resultado dessa bagunça:

```
kubectl get pods
``` 

Nesse ponto você pode ver que o Kubernetes está tentando realizar a atualização do nosso Deployment, mas não está conseguindo, pois no primeiro Pod que ele tentou atualizar, a probe falhou.

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-5fd6c688d8-kjf8d   0/1     Running   0          93s
nginx-deployment-fbdc9b65f-trnnz    1/1     Running   0          9m21s
nginx-deployment-fbdc9b65f-z8n4m    1/1     Running   0          9m21s
nginx-deployment-fbdc9b65f-zn8zh    1/1     Running   0          9m21s
``` 

Vamos ver o nosso rollout:

```
kubectl rollout status deployment/nginx-deployment
``` 

Waiting for deployment "nginx-deployment" rollout to finish: 1 out of 3 new replicas have been updated...
 

Mesmo depois de algum tempo o nosso rollout não terminou, ele continua esperando a nossa probe passar.

Podemos ver os detalhes do Pod que está com problema:

```
kubectl describe pod nginx-deployment-5fd6c688d8-kjf8d
 

Events:
  Type     Reason     Age                   From               Message
  ----     ------     ----                  ----               -------
  Normal   Scheduled  4m4s                  default-scheduler  Successfully assigned default/nginx-deployment-5fd6c688d8-kjf8d to ip-192-168-8-176.ec2.internal
  Normal   Pulled     4m3s                  kubelet            Container image "nginx:1.19.2" already present on machine
  Normal   Created    4m3s                  kubelet            Created container nginx
  Normal   Started    4m3s                  kubelet            Started container nginx
  Warning  Unhealthy  34s (x22 over 3m44s)  kubelet            Readiness probe failed: HTTP probe failed with statuscode: 404
``` 

Eu somente colei a parte final da saída, que é a parte mais interessante para esse momento. É nessa parte que podemos ver que o nosso Pod não está saudável, e por isso o Kubernetes não está conseguindo atualizar o nosso Deployment.
