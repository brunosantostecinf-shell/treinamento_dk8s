O Deployment e o ReplicaSet

Vamos criar um Deployment com o nome de nginx-deployment e vamos criar 3 réplicas do Pod do nginx.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    app: nginx-deployment
  name: nginx-deployment
spec:
  replicas: 1
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
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
``` 

Vamos visualizar o Deployment foi criado.

```
kubectl get deployments
``` 

A nossa saída será parecida com essa.

```
NAME               READY   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   1/1     1            1           7s
``` 

Simples, eu nós já sabiamos! Jeferson, eu quero saber sobre o ReplicaSet!

Calma, pois o nosso querido Deployment já criou o ReplicaSet para nós.

Vamos visualizar o ReplicaSet que foi criado.

```
kubectl get replicasets
```

A nossa saída será parecida com essa.

```
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   1         1         1       37s
``` 

Um coisa importante de observar na saída acima é que o ReplicaSet tem o mesmo nome do Deployment seguido de um sufixo aleatório, e ainda nessa saída podemos saber que o ReplicaSet atualmente tem 1 réplica do Pod do nginx rodando, de acordo com o que nós definimos no Deployment.

Vamos aumentar o número de réplicas do Pod do nginx para 3.

```
kubectl scale deployment nginx-deployment --replicas=3
``` 

Essa é uma forma de aumentar o número de réplicas do Pod do nginx sem precisar editar o Deployment, eu não recomendo, eu prefiro editar o Deployment e fazer o apply novamente, mas isso é uma questão de gosto e organização. Eu não gosto da ideia de ter que ficar fazendo scale no Deployment para aumentar ou diminuir o número de réplicas do Pod do nginx, sem ter isso registrado no git, por exemplo.

Alterando o Deployment para 3 réplicas.

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
      - image: nginx
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
```  

Pronto, você conhece as duas opções de aumentar o número de réplicas do Pod do nginx, fique a vonta para escolher a que você achar melhor.

Eu vou continuar usando a opção de editar o Deployment e fazer o apply novamente.

``` 
kubectl apply -f nginx-deployment.yaml
```  

Vamos verificar o nosso ReplicaSet novamente.

``` 
kubectl get replicasets
``` 

A nossa saída será parecida com essa.

``` 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       5m24s
```  

Perceba que o nome do ReplicaSet continua o mesmo, mas o número de réplicas mudou para 3. Quando somente alteramos o número de réplicas do nosso Deployment, o ReplicaSet permanece o mesmo, afinal sua principal função é fazer o gerenciamento das réplicas do Pod do nginx.

Agora vamos mudar a versão do nginx para a versão 1.19.2.

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
```  

Vamos aplicar as alterações.

``` 
kubectl apply -f nginx-deployment.yaml
```  

Pronto, agora o Deployment está usando a versão 1.19.2 do nginx.

Vamos verificar o nosso ReplicaSet novamente.

``` 
kubectl get replicasets
```  

Teremos agora a seguinte saída.

``` 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   0         0         0       8m53s
nginx-deployment-7858bcf56f   1         1         1       13s
```  

Agora sim nós temos um novo ReplicaSet com o nome de nginx-deployment-7858bcf56f, e o ReplicaSet antigo com o nome de nginx-deployment-6dd8d7cfbd foi esvaziado, pois ele não faz mais parte do Deployment, mas permanece no cluster, pois ele pode ser usado para fazer o rollback para a versão anterior do nginx.

Vamos ver um detalhe interessante em nosso Deployment.

``` 
kubectl describe deployment nginx-deployment
```  

Olha a linha referente ao ReplicaSet que está sendo gerenciado pelo Deployment.

``` 
NewReplicaSet:   nginx-deployment-7858bcf56f (3/3 replicas created)
```  

Sim, na saída do describe nós podemos ver que o Deployment está gerenciando o ReplicaSet com o nome de nginx-deployment-7858bcf56f e que ele tem 3 réplicas do Pod do nginx rodando.

Se você quiser fazer o rollback para a versão anterior do nginx, basta fazer o seguinte, conforme nós já vimos anteriormente.

``` 
kubectl rollout undo deployment nginx-deployment
``` 

Com isso será feito o rollback para a versão anterior do nginx e o ReplicaSet com o nome de nginx-deployment-7858bcf56f será esvaziado e o ReplicaSet com o nome de nginx-deployment-6dd8d7cfbd será preenchido novamente com 3 réplicas do Pod do nginx.

Vamos listar novamente os nossos ReplicaSet.

``` 
kubectl get replicasets
```  

O que temos agora é isso.

``` 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       15m
nginx-deployment-7858bcf56f   0         0         0       6m28s
```  

E se olharmos o Deployment novamente.

``` 
kubectl describe deployment nginx-deployment
``` 

Teremos a seguinte saída.

``` 
NewReplicaSet:   nginx-deployment-6dd8d7cfbd (3/3 replicas created)
```  

Simples demais, não é mesmo?

Então agora você já sabe como fazer o gerenciamento de réplicas do Pod do nginx usando o Deployment, e por consequência o ReplicaSet.