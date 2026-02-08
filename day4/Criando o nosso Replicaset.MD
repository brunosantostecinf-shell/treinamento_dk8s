Criando um ReplicaSet

Como foi falado anteriormente, é possível você criar um ReplicaSet sem ser através de um Deployment, apesar de insistir, não faça isso, pois o Deployment é a forma mais fácil de gerenciar os ReplicaSet e a saúde dos Pods.

Mas vamos lá, caso você queira criar um ReplicaSet sem ser através de um Deployment, basta fazer o seguinte.

Para o nosso exemplo, vamos criar um arquivo com o nome de nginx-replicaset.yaml e vamos colocar o seguinte conteúdo.

```yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
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

Antes de aplicar o nosso manifesto, vamos entender o que estamos fazendo. Se você observar o nosso arquivo, não tem nada de novo, ou seja, nada que você já não tenha aprendido até agora. A principal diferença é que agora nós estamos usando o kind: ReplicaSet e não o kind: Deployment, até o APIVersion é a mesmo.

Agora vamos aplicar o nosso manifesto.

``` 
kubectl apply -f nginx-replicaset.yaml
``` 

A saída será essa.

``` 
NAME                          DESIRED   CURRENT   READY   AGE
nginx-deployment-6dd8d7cfbd   3         3         3       21m
nginx-deployment-7858bcf56f   0         0         0       12m
nginx-replicaset              3         3         3       6s
```  

Agora nós temos 3 ReplicaSet sendo gerenciados pelo Kubernetes, sendo que 2 deles são gerenciados pelo Deployment e o outro foi o que acabamos de criar.

Vamos listar os Pods que estão rodando.

``` 
kubectl get pods
```  

Tudo rolando magicamente, certo?

Agora vamos fazer um teste, vamos alterar a versão do nginx para a versão 1.19.3, para isso vamos editar o nosso arquivo nginx-replicaset.yaml e vamos alterar a versão do nginx para a versão 1.19.3.

```yaml 
apiVersion: apps/v1
kind: ReplicaSet
metadata:
  labels:
    app: nginx-app
  name: nginx-replicaset
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx-app
  template:
    metadata:
      labels:
        app: nginx-app
    spec:
      containers:
      - image: nginx:1.19.3
        name: nginx
        resources:
          limits:
            cpu: "0.5"
            memory: 256Mi
          requests:
            cpu: 0.25
            memory: 128Mi
 ``` 

Pronto, agora vamos aplicar as alterações.

``` 
kubectl apply -f nginx-replicaset.yaml
``` 

Vamos listar os Pods novamente.

``` 
kubectl get pods
```  

Perceba que nada aconteceu, ou seja, o ReplicaSet não fez o rollout da nova versão do nginx, isso acontece porque o ReplicaSet não faz o gerenciamento de versões, ele apenas garante que o número de réplicas do Pod esteja sempre ativo.

Se você olhar os detalhes do ReplicaSet você vai ver que ele está gerenciando 3 réplicas do Pod e que a imagem do nginx é a versão 1.19.3, porém ele não recriou os Pods com a nova versão do nginx, ele somente irá fazer isso se você apagar os Pods manualmente ou se o Pod morrer por algum motivo.

kubectl describe replicaset nginx-replicaset
 

Agora vamos apagar um dos Pods para que o ReplicaSet crie um novo Pod com a nova versão do nginx.

``` 
kubectl delete pod nginx-replicaset-8r6md
```  

Agora o ReplicaSet vai criar um novo Pod com a nova versão do nginx, e gerando um problema para nós, pois agora nós temos duas versões do nginx rodando no nosso cluster.

``` 
kubectl get pods -o=jsonpath='{range .items[*]}{"\n"}{.metadata.name}{"\t"}{range .spec.containers[*]}{.image}{"\t"}{end}{end}'
``` 

Essa é uma forma de você listar os Pods e as imagens que estão sendo usadas por eles, eu sei que é bastante esquisito, mas vou explicar o que está acontecendo.

- kubectl get pods: esse comando lista todos os Pods no cluster.

- -o=jsonpath: esse parâmetro especifica que queremos usar a saída em formato JSONPath para exibir as informações dos Pods.

- '{range .items[*]}{"\n"}{.metadata.name}{"\t"}{range .spec.containers[*]}{.image}{"\t"}{end}{end}': essa é a expressão JSONPath que define o formato de saída do comando. Ela usa a função range para iterar sobre todos os objetos items (ou seja, os Pods) retornados pelo comando kubectl get pods. Em seguida, exibe o nome do Pod ({.metadata.name}) seguido de uma tabulação (\t), e itera sobre todos os contêineres ({range .spec.containers[*]}) dentro do Pod, exibindo a imagem usada por cada um deles ({.image}). Por fim, insere uma quebra de linha (\n) e fecha o segundo range com {end}{end}.

Sim, eu sei, continua confuso!

Mas vou te contar um segredo, somente com o tempo e utilizando repetidas vezes, as coisas começam a ficar mais fáceis, então não desista! Pra trás, nem pra pegar impulso!

Ainda vamos falar com mais detalhes sobre como utilizar metadata para ter uma saída mais amigável e precisa.

Apagando o ReplicaSet

Para remover o ReplicaSet e todos os Pods que ele está gerenciando, basta executar o comando abaixo.

``` 
kubectl delete replicaset nginx-replicaset
```  

Caso você queira fazer isso utilizando o arquivo de manifesto, basta executar o comando abaixo.

``` 
kubectl delete -f nginx-replicaset.yaml
``` 

Pronto, o nosso ReplicaSet foi removido e todos os Pods que ele estava gerenciando também foram removidos.

Durante a nossa sessão, nós já aprendemos como criar um ReplicaSet e como ele funciona, mas ainda temos muito o que aprender, então vamos continuar.

