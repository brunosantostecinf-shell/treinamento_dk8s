PVC - Persistent Volume Claim

O PVC é uma solicitação de armazenamento feita pelos usuários ou aplicativos no cluster Kubernetes. Ele permite que os usuários solicitem um volume específico, com base em tamanho, tipo de armazenamento e outras características. O PVC age como uma "assinatura" que reivindica um PV para ser usado por um contêiner. O Kubernetes tenta associar automaticamente um PVC a um PV compatível, garantindo que o armazenamento seja alocado corretamente.

Através do PVC, as pessoas podem abstrair os detalhes de cada tipo de armazenamento, permitindo maior flexibilidade e portabilidade entre diferentes ambientes e provedores de infraestrutura. Ele também permite que os usuários solicitem volumes com diferentes características, como tamanho, tipo de armazenamento e modo de acesso.

Todo PVC é associado a um Storage Class ou a um Persistent Volume. O Storage Class é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Já o Persistent Volume é um recurso que representa um volume de armazenamento disponível para ser usado pelo cluster.

Vamos criar o nosso primeiro PVC para o PV que criamos anteriormente.

Para isso, vamos criar um arquivo chamado pvc.yaml e adicionar o seguinte conteúdo:

```yaml
apiVersion: v1 # versão da API do Kubernetes
kind: PersistentVolumeClaim # tipo de recurso, no caso, um PersistentVolumeClaim
metadata: # metadados do recurso
  name: meu-pvc # nome do PVC
spec: # especificação do PVC
  accessModes: # modo de acesso ao volume
    - ReadWriteOnce # modo de acesso RWO, ou seja, somente leitura e escrita por um nó
  resources: # recursos do PVC
    requests: # solicitação de recursos
      storage: 1Gi # tamanho do volume que ele vai solicitar
  storageClassName: nfs # nome da classe de armazenamento que será utilizada
  selector: # seletor de labels
    matchLabels: # labels que serão utilizadas para selecionar o PV
      storage: nfs # label que será utilizada para selecionar o PV
``` 

Aqui nós estamos definindo o nosso PVC, e vou falar um pouco sobre as principais seções do nosso arquivo.

A seção accessModes é onde definimos o modo de acesso ao volume, que pode ser ReadWriteOnce (RWO), ReadOnlyMany (ROM) ou ReadWriteMany (RWM). O RWO significa que o volume pode ser montado como somente leitura e escrita por um nó. O ROM significa que o volume pode ser montado como somente leitura por vários nós. O RWM significa que o volume pode ser montado como leitura e escrita por vários nós.

A seção resources é onde definimos os recursos que o PVC irá solicitar. Nesse caso, estamos solicitando um volume de 1Gi.

Ainda temos a seção storageClassName, que é onde definimos o nome da classe de armazenamento que iremos associar ao PVC.

E por fim, temos a seção selector, que é onde definimos o seletor de labels que será utilizado para selecionar o PV que será associado ao PVC.

Vamos criar o nosso PVC.

```
kubectl apply -f pvc.yaml
persistentvolumeclaim/meu-pvc created
``` 

Pronto, PVC criado! Vamos conferir se ele foi criado corretamente.

```
kubectl get pvc 

NAME      STATUS    VOLUME   CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Pending                                      nfs            5s
``` 

Está lá! Porém o status dele está como Pending, vamos ver se tem alguma informação que nos ajude a entender o que está acontecendo.

```
kubectl describe pvc meu-pvc
 
Name:          meu-pvc
Namespace:     default
StorageClass:  nfs
Status:        Pending
Volume:        
Labels:        <none>
Annotations:   <none>
Finalizers:    [kubernetes.io/pvc-protection]
Capacity:      
Access Modes:  
VolumeMode:    Filesystem
Used By:       <none>
Events:
  Type    Reason                Age                 From                         Message
  ----    ------                ----                ----                         -------
  Normal  WaitForFirstConsumer  15s (x4 over 1m5s)  persistentvolume-controller  waiting for first consumer to be created before binding
``` 

Repare na parte dos eventos, lá diz que o PVC está esperando o primeiro consumidor ser criado antes de ser vinculado. O que isso significa?

Significa que o PVC está esperando que um Pod seja criado para que ele possa ser vinculado ao PV, então bora criar o nosso Pod.

Vamos usar o nosso já conhecido Nginx como exemplo, então vamos criar um arquivo chamado pod.yaml e adicionar o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
  - name: nginx
    image: nginx:latest
    ports:
    - containerPort: 80
    volumeMounts:
    - name: meu-pvc
      mountPath: /usr/share/nginx/html
  volumes:
  - name: meu-pvc
    persistentVolumeClaim:
      claimName: meu-pvc
``` 

O que estamos fazendo aqui é basicamente o seguinte:

- Criando um Pod com o nome nginx-pod;
- Utilizando a imagem nginx:latest como base;
- Expondo a porta 80;
- Definindo um volume chamado meu-pvc e montando ele no caminho /usr/share/nginx/html dentro do container;
- Por fim, definindo que o volume meu-pvc é um PersistentVolumeClaim e que o nome do PVC é meu-pvc.

Esse trecho do arquivo pod.yaml é responsável por montar o volume meu-pvc no caminho /usr/share/nginx/html dentro do container.

```
    volumeMounts: # montando o volume no container
    - name: meu-pvc  # nome do volume
      mountPath: /usr/share/nginx/html # caminho onde o volume será montado no container
  volumes: # definindo o volume que será utilizado pelo Pod
  - name: meu-pvc # nome do volume
    persistentVolumeClaim: # tipo de volume, no caso, um PersistentVolumeClaim
      claimName: meu-pvc # nome do PVC
```

Esclarecido! Vamos criar o nosso Pod.

```
kubectl apply -f pod.yaml
pod/nginx-pod created
``` 

Bora ver se está tudo certo com o nosso Pod.

```
NAME        READY   STATUS    RESTARTS   AGE
nginx-pod   1/1     Running   0          21s
``` 

Parece que sim! Agora vamos ver se o nosso PVC foi vinculado ao PV.

```
kubectl get pvc
 

NAME      STATUS   VOLUME       CAPACITY   ACCESS MODES   STORAGECLASS   AGE
meu-pvc   Bound    meu-pv-nfs   1Gi        RWO            nfs            3m8s
``` 

Opa! Temos um vinculo!

Vamos ver se tem alguma coisa nova na saída do get pv.

```
kubectl get pv
 

NAME         CAPACITY   ACCESS MODES   RECLAIM POLICY   STATUS   CLAIM             STORAGECLASS   REASON   AGE
meu-pv-nfs   1Gi        RWO            Retain           Bound    default/meu-pvc   nfs                     3m42s
``` 

Agora sim! Temos um PV com o status Bound e um PVC com o status Bound também. \o/

E pra finalizar o nosso primeiro teste, vamos ver se o nosso Pod está utilizando o nosso volume.

```
kubectl describe pod nginx-pod
 

Name:             nginx-pod
Namespace:        default
Priority:         0
Service Account:  default
Node:             kind-linuxtips-worker/172.18.0.4
Start Time:       Tue, 11 Apr 2023 01:47:48 +0200
Labels:           <none>
Annotations:      <none>
Status:           Running
IP:               10.244.2.3
IPs:
  IP:  10.244.2.3
Containers:
  nginx:
    Container ID:   containerd://b5946958f63c392c8a77b06811f7859113a1dd260ebcc2113579af6b32c4f549
    Image:          nginx:latest
    Image ID:       docker.io/library/nginx@sha256:2ab30d6ac53580a6db8b657abf0f68d75360ff5cc1670a85acb5bd85ba1b19c0
    Port:           80/TCP
    Host Port:      0/TCP
    State:          Running
      Started:      Tue, 11 Apr 2023 01:47:50 +0200
    Ready:          True
    Restart Count:  0
    Environment:    <none>
    Mounts:
      /usr/share/nginx/html from meu-pvc (rw)
      /var/run/secrets/kubernetes.io/serviceaccount from kube-api-access-8874f (ro)
Conditions:
  Type              Status
  Initialized       True 
  Ready             True 
  ContainersReady   True 
  PodScheduled      True 
Volumes:
  meu-pvc:
    Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
    ClaimName:  meu-pvc
    ReadOnly:   false
  kube-api-access-8874f:
    Type:                    Projected (a volume that contains injected data from multiple sources)
    TokenExpirationSeconds:  3607
    ConfigMapName:           kube-root-ca.crt
    ConfigMapOptional:       <nil>
    DownwardAPI:             true
QoS Class:                   BestEffort
Node-Selectors:              <none>
Tolerations:                 node.kubernetes.io/not-ready:NoExecute op=Exists for 300s
                             node.kubernetes.io/unreachable:NoExecute op=Exists for 300s
Events:
  Type    Reason     Age   From               Message
  ----    ------     ----  ----               -------
  Normal  Scheduled  8s    default-scheduler  Successfully assigned default/nginx-pod to kind-linuxtips-worker
  Normal  Pulling    7s    kubelet            Pulling image "nginx:latest"
  Normal  Pulled     6s    kubelet            Successfully pulled image "nginx:latest" in 799.112685ms (799.119928ms including waiting)
  Normal  Created    6s    kubelet            Created container nginx
  Normal  Started    6s    kubelet            Started container nginx
``` 

Pronto! O nosso Pod está utilizando o nosso volume! Todo o conteúdo que for criado dentro do Pod será armazenado no nosso volume, e mesmo que o Pod seja removido, o conteúdo não será perdido.

Agora vamos testar o nosso volume. Vamos criar um arquivo HTML simples no diretório /mnt/data do nosso servidor NFS.

```bash
echo "<h1>GIROPOPS STRIGUS GIRUS</h1>" > /mnt/data/index.html
``` 

Agora vamos ver se o nosso arquivo foi criado.

```
kubectl exec -it nginx-pod -- ls /usr/share/nginx/html 

index.html
``` 

Está lá! Vamos dar um curl de dentro do Pod para ver se o Ngix está servindo o nosso arquivo.

```
kubectl exec -it nginx-pod -- curl localhost

<h1>GIROPOPS STRIGUS GIRUS</h1>
```

Tudo rolando maravilhosamente bem! :D

