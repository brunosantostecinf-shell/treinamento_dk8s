### O que são Tolerations?

Agora que entendemos como os Taints funcionam e como eles influenciam o agendamento de Pods nos Nodes, vamos mergulhar no mundo das Tolerations. As Tolerations são como o "antídoto" para os Taints. Elas permitem que um Pod seja agendado em um Node que possui um Taint específico. Em outras palavras, elas "toleram" as Taints.

Vamos voltar ao nosso cluster da Strigus para entender melhor como isso funciona.

Imagine que temos um workload crítico que precisa ser executado em um Node com GPU. Já marcamos nossos Nodes com GPU com a label gpu=true, e agora vamos usar Tolerations para garantir que nosso Pod possa ser agendado nesses Nodes. Isso não faz com que o Pod seja agendado nesses Nodes, mas permite que ele seja agendado nesses Nodes. Entendeu a diferença?

Primeiro, vamos criar um Taint no Node strigus-worker1, que possui uma GPU.

```bash
kubectl taint nodes strigus-worker1 gpu=true:NoSchedule
```

Com esse Taint, estamos dizendo que nenhum Pod será agendado nesse Node, a menos que ele tenha uma Toleration específica para a Taint gpu=true.

Vamos criar um Deployment com 5 réplicas do Nginx e ver o que acontece.

```bash
kubectl create deployment nginx --image=nginx --replicas=5
```

Agora vamos verificar se os Pods foram criados e se estão em execução.

```bash
NAME                     READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-77b4fdf86c-274jk   1/1     Running   0          5s    10.244.6.31   strigus-worker2   <none>           <none>
nginx-77b4fdf86c-97r8d   1/1     Running   0          5s    10.244.5.25   strigus-worker4   <none>           <none>
nginx-77b4fdf86c-cm96h   1/1     Running   0          5s    10.244.6.30   strigus-worker2   <none>           <none>
nginx-77b4fdf86c-rhdmh   1/1     Running   0          5s    10.244.4.29   strigus-worker3   <none>           <none>
nginx-77b4fdf86c-ttqzg   1/1     Running   0          5s    10.244.4.30   strigus-worker3   <none>           <none>
```

Como esperado, nenhum Pod foi agendado no Node strigus-worker1, que é o Node que aplicamos o Taint.

Agora, vamos modificar o nosso Deployment do Nginx para que ele tenha uma Toleration para a Taint gpu=true.

Para ficar mais fácil, vamos criar um manifesto para o nosso Deployment utilizando o comando kubectl create deployment nginx --image=nginx --replicas=5 --dry-run=client -o yaml > gpu-deployment.yaml.

Agora vamos editar o arquivo gpu-deployment.yaml e adicionar a Toleration para a Taint gpu=true.

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
      tolerations:
      - key: gpu
        operator: Equal
        value: "true"
        effect: NoSchedule
```
Vamos entender o que adicionamos no arquivo.

```yaml
      tolerations:
      - key: gpu
        operator: Equal
        value: "true"
        effect: NoSchedule
```
Onde:

key é a chave do Taint que queremos tolerar.
operator é o operador que queremos utilizar. Nesse caso, estamos utilizando o operador Equal, que significa que o valor do Taint precisa ser igual ao valor da Toleration.
value é o valor do Taint que queremos tolerar.
effect é o efeito do Taint que queremos tolerar. Nesse caso, estamos utilizando o efeito NoSchedule, que significa que o Kubernetes não irá agendar nenhum Pod nesse Node a menos que ele tenha uma Toleration correspondente.
Vamos aplicar esse Deployment e ver o que acontece.

```bash
kubectl apply -f gpu-deployment.yaml
```
Se verificarmos os Pods agora, veremos que o nosso Pod gpu-pod está rodando no Node strigus-worker1.

```bash
kubectl get pods -o wide
```

A saída mostrará algo como:

```bash
NAME                    READY   STATUS    RESTARTS   AGE   IP            NODE              NOMINATED NODE   READINESS GATES
nginx-7b68fffb4-czrpt   1/1     Running   0          11s   10.244.5.24   strigus-worker4   <none>           <none>
nginx-7b68fffb4-d577x   1/1     Running   0          11s   10.244.4.28   strigus-worker3   <none>           <none>
nginx-7b68fffb4-g2kxr   1/1     Running   0          11s   10.244.3.10   strigus-worker1    <none>           <none>
nginx-7b68fffb4-m5kln   1/1     Running   0          11s   10.244.6.29   strigus-worker2   <none>           <none>
nginx-7b68fffb4-n6kck   1/1     Running   0          11s   10.244.3.11   strigus-worker1    <none>           <none>
```
Isso demonstra o poder das Tolerations em combinação com os Taints. Podemos controlar com precisão onde nossos Pods são agendados, garantindo que workloads críticos tenham os recursos que necessitam.

Para remover o Taint do Node strigus-worker1, basta usar o comando kubectl taint nodes strigus-worker1 gpu=true:NoSchedule-.

Mas lembrando mais uma vez, as Tolerations não fazem com que o Pod seja agendado nesses Nodes, mas permite que ele seja agendado nesses Nodes.

Então, caso você queira garantir que determinado Pod seja executado em determinado Node, você precisa utilizar o conceito de Affinity, que veremos agora.