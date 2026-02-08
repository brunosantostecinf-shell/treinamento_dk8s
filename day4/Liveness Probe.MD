Liveness Probe

A livenessProbe é a nossa probe de verificação de integridade, o que ela faz é verificar se o que está rodando dentro do Pod está saudável. O que fazemos é criar uma forma de testar se o que temos dentro do Pod está respondendo conforme esperado. Se por acaso o teste falhar, o Pod será reiniciado.

Para ficar mais claro, vamos mais uma vez utilizar o exemplo com o Nginx. Gosto de usar o Nginx como exemplo, pois sei que toda pessoa já o conhece, e assim, fica muito mais fácil de entender o que está acontecendo. Afinal, você está aqui para aprender Kubernetes, e se for com algo que você já conhece, fica muito mais fácil de entender.

Bem, vamos lá, hora de criar um novo Deployment com o Nginx, vamos utilizar o exemplo que já utilizamos quando aprendemos sobre o Deployment.

Para isso, crie um arquivo chamado nginx-liveness.yaml e cole o seguinte conteúdo.

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
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          tcpSocket: # Aqui vamos utilizar o tcpSocket, onde vamos se conectar ao container através do protocolo TCP
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
``` 

Com isso temos algumas coisas novas, e utilizamos apenas uma probe que é a livenessProbe.

O que declaramos com a regra acima é que queremos testar se o Pod está respondendo através do protocolo TCP, através da opção tcpSocket, na porta 80 que foi definida pela opção port. E também definimos que queremos esperar 10 segundos para executar a primeira verificação utilizando initialDelaySeconds e por conta da periodSecondsfalamos que queremos que a cada 10 segundos seja realizada a verificação. Caso a verificação falhe, vamos esperar 5 segundos, por conta da timeoutSeconds, para tentar novamente, e como utilizamos o failureThreshold, se falhar mais 3 vezes, vamos reiniciar o Pod.

Ficou mais claro? Vamos para mais um exemplo.

Vamos imaginar que agora não queremos mais utilizar o tcpSocket, mas sim o httpGet para tentar acessar um endpoint dentro do nosso Pod.

Para isso, vamos alterar o nosso nginx-deployment.yaml para o seguinte.

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
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          httpGet: # Aqui vamos utilizar o httpGet, onde vamos se conectar ao container através do protocolo HTTP
            path: / # Qual o endpoint que vamos utilizar para se conectar ao container
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
``` 

Perceba que agora somente mudamos algumas coisas, apesar de seguir com o mesmo objetivo, que é verificar se o Nginx está respondendo corretamente, mudamos como iremos testar isso. Agora estamos utilizando o httpGet para testar se o Nginx está respondendo corretamente através do protocolo HTTP, e para isso, estamos utilizando o endpoint / e a porta 80.

O que temos de novo aqui é a opção path, que é o endpoint que vamos utilizar para testar se o Nginx está respondendo corretamente, e claro, a httpGet é a forma como iremos realizar o nosso teste, através do protocolo HTTP.

 

Escolha qual dois dois exemplos você quer utilizar, e crie o seu Deployment através do comando abaixo.

```
kubectl apply -f nginx-deployment.yaml
``` 

Para verificar se o Deployment foi criado corretamente, execute o comando abaixo.

```
kubectl get deployments
``` 

Você deve ver algo parecido com isso.

```
NAME                                READY   STATUS    RESTARTS   AGE
nginx-deployment-7557d7fc6c-dx48d   1/1     Running   0          14s
nginx-deployment-7557d7fc6c-tbk4w   1/1     Running   0          12s
nginx-deployment-7557d7fc6c-wv876   1/1     Running   0          16s
``` 

Para que você possa ver mais detalhes sobre o seu Pod e saber se a nossa probe está funcionando corretamente, vamos utilizar o comando abaixo.

```
kubectl describe pod nginx-deployment-7557d7fc6c-dx48d
``` 

A saída deve ser parecida com essa.

```
Name:             nginx-deployment-589d6fc888-42fmg
Namespace:        default
Priority:         0
Service Account:  default
Node:             ip-192-168-39-119.ec2.internal/192.168.39.119
Start Time:       Thu, 16 Mar 2023 18:49:53 +0100
Labels:           app=nginx-deployment
                  pod-template-hash=589d6fc888
Annotations:      kubernetes.io/psp: eks.privileged
Status:           Running
IP:               192.168.49.40
IPs:
  IP:           192.168.49.40
Controlled By:  ReplicaSet/nginx-deployment-589d6fc888
Containers:
  nginx:
    Container ID:   docker://f7fc28a1fafbf53471ba144d4fb48bc029d289d93b3565b839ae89a1f38cd894
    Image:          nginx:1.19.2
    Image ID:       docker-pullable://nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 16 Mar 2023 18:49:59 +0100
    Ready:          True
    Restart Count:  0
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Liveness:     http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8srlq (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-8srlq:
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
  Normal  Scheduled  21s   default-scheduler  Successfully assigned default/nginx-deployment-589d6fc888-42fmg to ip-192-168-39-119.ec2.internal
  Normal  Pulling    20s   kubelet            Pulling image "nginx:1.19.2"
  Normal  Pulled     15s   kubelet            Successfully pulled image "nginx:1.19.2" in 4.280120301s (4.280125621s including waiting)
  Normal  Created    15s   kubelet            Created container nginx
  Normal  Started    15s   kubelet            Started container nginx
``` 

Aqui temos a informação mais importante para nós nesse momento:

```
    Liveness:     http-get http://:80/ delay=10s timeout=5s period=10s #success=1 #failure=3
``` 

A saída acima é parte da saída do comando kubectl describe pod. Tudo funcionando maravilhosamente bem.

Agora vamos fazer o seguinte, vamos alterar o nosso Deployment, para que a nossa probe falhe. Para isso vamos alterar o endpoint que estamos utilizando. Vamos alterar o path para /giropops.

 

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
        livenessProbe: # Aqui é onde vamos adicionar a nossa livenessProbe
          httpGet: # Aqui vamos utilizar o httpGet, onde vamos se conectar ao container através do protocolo HTTP
            path: /giropops # Qual o endpoint que vamos utilizar para se conectar ao container
            port: 80 # Qual porta TCP vamos utilizar para se conectar ao container
          initialDelaySeconds: 10 # Quantos segundos vamos esperar para executar a primeira verificação
          periodSeconds: 10 # A cada quantos segundos vamos executar a verificação
          timeoutSeconds: 5 # Quantos segundos vamos esperar para considerar que a verificação falhou
          failureThreshold: 3 # Quantos falhas consecutivas vamos aceitar antes de reiniciar o container
``` 

Vamos aplicar as alterações no nosso Deployment:

```
kubectl apply -f deployment.yaml
``` 

Depois de um tempo, você perceberá que o Kubernetes finalizou a atualização do nosso Deployment. Se você aguardar um pouco mais, você irá perceber que os Pods estã̀o sendo reiniciados com frequência.

Tudo isso porque a nossa livenessProbe está falhando, afinal o nosso endpoint está errado.

Podemos ver mais detalhes sobre o que está acontecendo na saída do comando kubectl describe pod:

```
kubectl describe pod nginx-deployment-7557d7fc6c-dx48d
 

Name:             nginx-deployment-7557d7fc6c-dx48d
Namespace:        default
Priority:         0
Service Account:  default
Node:             ip-192-168-39-119.ec2.internal/192.168.39.119
Start Time:       Thu, 16 Mar 2023 18:51:00 +0100
Labels:           app=nginx-deployment
                  pod-template-hash=7557d7fc6c
Annotations:      kubernetes.io/psp: eks.privileged
Status:           Running
IP:               192.168.44.84
IPs:
  IP:           192.168.44.84
Controlled By:  ReplicaSet/nginx-deployment-7557d7fc6c
Containers:
  nginx:
    Container ID:   docker://c070d9c08bec40ad14562512d7bd8507a44279a327f1b3ecac1621da7ccf21b4
    Image:          nginx:1.19.2
    Image ID:       docker-pullable://nginx@sha256:c628b67d21744fce822d22fdcc0389f6bd763daac23a6b77147d0712ea7102d0
    Port:           <none>
    Host Port:      <none>
    State:          Running
      Started:      Thu, 16 Mar 2023 18:51:41 +0100
    Last State:     Terminated
      Reason:       Completed
      Exit Code:    0
      Started:      Thu, 16 Mar 2023 18:51:02 +0100
      Finished:     Thu, 16 Mar 2023 18:51:40 +0100
    Ready:          True
    Restart Count:  1
    Limits:
      cpu:     500m
      memory:  256Mi
    Requests:
      cpu:        250m
      memory:     128Mi
    Liveness:     http-get http://:80/giropops delay=10s timeout=5s period=10s #success=1 #failure=3
    Environment:  <none>
    Mounts:
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-4sk2f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  kube-api-access-4sk2f:
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
  Type     Reason     Age               From               Message
  ----     ------     ----              ----               -------
  Normal   Scheduled  44s               default-scheduler  Successfully assigned default/nginx-deployment-7557d7fc6c-dx48d to ip-192-168-39-119.ec2.internal
  Normal   Pulled     4s (x2 over 43s)  kubelet            Container image "nginx:1.19.2" already present on machine
  Normal   Created    4s (x2 over 43s)  kubelet            Created container nginx
  Warning  Unhealthy  4s (x3 over 24s)  kubelet            Liveness probe failed: HTTP probe failed with statuscode: 404
  Normal   Killing    4s                kubelet            Container nginx failed liveness probe, will be restarted
  Normal   Started    3s (x2 over 42s)  kubelet            Started container nginx
``` 

Na última parte da saída do comando kubectl describe pod, você pode ver que o Kubernetes está tentando executar a nossa livenessProbe e ela está falhando, inclusive ele mostra a quantidade de vezes que ele tentou executar a livenessProbe e falhou, e com isso, ele reiniciou o nosso Pod.

 

Acho que agora ficou bem mais claro como a livenessProbe funciona, então é hora de partir para a próxima probe, a readinessProbe.
