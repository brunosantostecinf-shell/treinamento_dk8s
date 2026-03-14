### O que são Affinity e Antiaffinity?

Affinity e Antiaffinity são conceitos que permitem que você defina regras para o agendamento de Pods em determinados Nodes. Com eles você pode definir regras para que Pods sejam agendados em Nodes específicos, ou até mesmo para que Pods não sejam agendados em Nodes específicos.

Vamos entender como isso funciona na prática.

Você se lembra que adicionamos a label gpu=true nos Nodes que possuem GPU? Então, vamos utilizar essa label para garantir que o nosso Pod seja agendado somente neles. Para isso, vamos utilizar o conceito de Affinity.

Vamos criar um Deployment com 5 réplicas do Nginx com a seguinte configuração:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
```
Vamos entender o que temos de novo:

```yaml
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: gpu
                operator: In
                values:
                - "true"
```
Onde:

* affinity é o início da configuração de Affinity.
* nodeAffinity é o conceito de Affinity para Nodes.
* requiredDuringSchedulingIgnoredDuringExecution é usado para indicar que o Pod só pode ser agendado em um Node que atenda aos requisitos de Affinity, está falando que essa regra é obrigatória no momento do agendamento do Pod, mas que pode ser ignorada durante a execução do Pod.
* nodeSelectorTerms é usado para indicar os termos de seleção de Nodes, que será usado para selecionar os Nodes que atendem aos requisitos de Affinity.
* matchExpressions é usado para indicar as expressões de seleção de Nodes, ou seja, o nome da label, o operador e o valor da label.
* key é o nome da label que queremos utilizar para selecionar os Nodes.
* operator é o operador que queremos utilizar. Nesse caso, estamos utilizando o operador In, que significa que o valor da label precisa estar dentro dos valores que estamos informando.
* values é o valor da label que queremos utilizar para selecionar os Nodes.

Sendo assim, estamos falando para o Kubernetes que o nosso Pod só pode ser agendado em um Node que tenha a label gpu=true. Simples assim!

Vamos aplicar esse Deployment e ver o que acontece.

```yaml
kubectl apply -f gpu-deployment.yaml
```

Se verificarmos os Pods agora, veremos que os nossos Pods estão rodando somente nos Nodes que possuem a label gpu=true.

```bash
kubectl get pods -o wide
```
A saída mostrará algo como:

```bash
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-5dd89c4b9b-hwzcx   1/1     Running   0          4s    10.244.3.13   strigus-worker1    <none>           <none>
nginx-5dd89c4b9b-m4fj2   1/1     Running   0          4s    10.244.5.37   strigus-worker4   <none>           <none>
nginx-5dd89c4b9b-msnv8   1/1     Running   0          4s    10.244.3.14   strigus-worker1    <none>           <none>
nginx-5dd89c4b9b-nlcgs   1/1     Running   0          4s    10.244.5.36   strigus-worker4   <none>           <none>
nginx-5dd89c4b9b-trgw7   1/1     Running   0          4s    10.244.3.12   strigus-worker1    <none>           <none>
```

Isso demonstra o poder do Affinity. Podemos controlar com precisão onde nossos Pods serão agendados, garantindo que workloads críticos tenham os recursos que necessitam. Sensacional demais!

Agora vamos entender o conceito de Antiaffinity.

### O Antiaffinity?

O Antiaffinity é o conceito contrário ao Affinity. Com ele você pode definir regras para que Pods não sejam agendados em Nodes específicos.

Vamos conhecer ele na prática!

Vamos relembrar como os nossos Nodes estão distribuidos.

* strigus-br-sp
    * strigus-br-sp-1
        * strigus-control-plane1
        * strigus-worker3
    * strigus-br-sp-2
        * strigus-control-plane4
        * strigus-worker1
* strigus-br-ssa
    * strigus-br-ssa-1
        * strigus-control-plane2
        * strigus-worker2
    * strigus-br-ssa-2
        * strigus-control-plane3
        * strigus-worker4
Ou seja, duas regiões e duas zonas de disponibilidade em cada região, certo?

Cada Node está com as labels region e datacenter devidamente configuradas, representando a região e a zona de disponibilidade que ele está.

Agora vamos imaginar que precisamos garantir que a nossa estará sendo distribuida entre as regiões e zonas de disponibilidade, ou seja, precisamos garantir que os nossos Pods não sejam agendados em Nodes que estejam na mesma região e na mesma zona de disponibilidade.

Para isso, vamos utilizar o conceito de Antiaffinity. \o/

Vamos criar um Deployment com 5 réplicas do Nginx com a seguinte configuração:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx
  name: nginx
spec:
  replicas: 5
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - image: nginx
        name: nginx
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "datacenter"
```

Vamos entender o que estamos fazendo:

```yaml
      affinity:
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
          - labelSelector:
              matchExpressions:
              - key: app
                operator: In
                values:
                - nginx
            topologyKey: "datacenter"
```
Onde:

* affinity é o início da configuração de Affinity.
* podAntiAffinity é o conceito de Antiaffinity para Pods.
* requiredDuringSchedulingIgnoredDuringExecution é usado para indicar que o Pod só pode ser agendado em um Node que atenda aos requisitos de Antiaffinity, está falando que essa regra é obrigatória no momento do agendamento do Pod, mas que pode ser ignorada durante a execução do Pod.
* labelSelector é usado para indicar os termos de seleção de Pods, que será usado para selecionar os Pods que atendem aos requisitos de Antiaffinity.
* matchExpressions é usado para indicar as expressões de seleção de Pods, ou seja, o nome da label, o operador e o valor da label.
* key é o nome da label que queremos utilizar para selecionar os Pods.
* operator é o operador que queremos utilizar. Nesse caso, estamos utilizando o operador In, que significa que o valor da label precisa estar dentro dos valores que estamos informando.
* values é o valor da label que queremos utilizar para selecionar os Pods.
* topologyKey é usado para indicar a chave de topologia que será usada para selecionar os Pods. Nesse caso, estamos utilizando a chave datacenter, que é a chave que representa a zona de disponibilidade.

Sendo assim, estamos falando para o Kubernetes que o nosso Pod só pode ser agendado em um Node que não tenha nenhum Pod com a label app=nginx na mesma zona de disponibilidade.

Você já deve estar imaginando que isso irá dar errado, certo? Pois somente temos 4 datacenters, e estamos tentando agendar 5 Pods, ou seja, um Pod não será agendado.

Vamos aplicar esse Deployment e ver o que acontece.

```bash
kubectl apply -f nginx-deployment.yaml
```
Se verificarmos os Pods agora, veremos que somente 4 Pods estão rodando, e um Pod não foi agendado.

```bash
kubectl get pods -o wide
```
A saída mostrará algo como:

```bash
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-58b9c8f764-7rqr7   1/1     Running   0          8s    10.244.4.32   strigus-worker3   <none>           <none>
nginx-58b9c8f764-mfpp7   1/1     Running   0          8s    10.244.6.33   strigus-worker2   <none>           <none>
nginx-58b9c8f764-n45p6   1/1     Running   0          8s    10.244.3.15   strigus-worker    <none>           <none>
nginx-58b9c8f764-qwjdk   0/1     Pending   0          8s    <none>        <none>            <none>           <none>
nginx-58b9c8f764-qzffs   1/1     Running   0          8s    10.244.5.38   strigus-worker4   <none>           <none>
```

Perceba que o Pod nginx-58b9c8f764-qwjdk está com o status Pending, pois não foi possível agendar ele em nenhum Node, pois em nossa regra nós falamos que ele não pode ser agendado em Nodes que tenham Pods com a label app=nginx na mesma zona de disponibilidade, ou seja, não teremos nenhum Node que atenda a essa regra.

Sensacional demais, eu sei! \o/