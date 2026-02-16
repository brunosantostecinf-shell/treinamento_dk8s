### Introdução ao Metrics Server

Antes de começarmos a explorar o Horizontal Pod Autoscaler (HPA), é essencial termos o Metrics Server instalado em nosso cluster Kubernetes. O Metrics Server é um agregador de métricas de recursos de sistema, que coleta métricas como uso de CPU e memória dos nós e pods no cluster. Essas métricas são vitais para o funcionamento do HPA, pois são usadas para determinar quando e como escalar os recursos.

### Por que o Metrics Server é importante para o HPA?

O HPA utiliza métricas de uso de recursos para tomar decisões inteligentes sobre o escalonamento dos pods. Por exemplo, se a utilização da CPU de um pod exceder um determinado limite, o HPA pode decidir aumentar o número de réplicas desse pod. Da mesma forma, se a utilização da CPU for muito baixa, o HPA pode decidir reduzir o número de réplicas. Para fazer isso de forma eficaz, o HPA precisa ter acesso a métricas precisas e atualizadas, que são fornecidas pelo Metrics Server. Portanto, precisamos antes conhecer essa peça fundamental para o dia de hoje! :D

### Instalando o Metrics Server

#### No Amazon EKS e na maioria dos clusters Kubernetes
Durante a nossa aula, estou com um cluster EKS, e para instalar o Metrics Server, podemos usar o seguinte comando:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

Esse comando aplica o manifesto do Metrics Server ao seu cluster, instalando todos os componentes necessários.

#### No Minikube:
A instalação do Metrics Server no Minikube é bastante direta. Use o seguinte comando para habilitar o Metrics Server:

```
minikube addons enable metrics-server
```

Após a execução deste comando, o Metrics Server será instalado e ativado em seu cluster Minikube.

#### No KinD (Kubernetes in Docker):
Para o KinD, você pode usar o mesmo comando que usou para o EKS:

```
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

É importante realizar o patch no deployment do Metrics-server da seguinte forma:

```
kubectl patch -n kube-system deployment metrics-server --type=json \
  -p '[{"op":"add","path":"/spec/template/spec/containers/0/args/-","value":"--kubelet-insecure-tls"}]'
```

Isso é importante para evitar problemas com o certificado TLS do kubelet.

### Realizando a instalação via Helm

Para que você possa ter o Metrics-Server instalado em seu cluster, você pode utilizar o Helm, que é um gerenciador de pacotes para Kubernetes.

Para isso basta executar os seguintes comandos:

```
helm repo add metrics-server https://kubernetes-sigs.github.io/metrics-server/
helm repo update
helm upgrade --install --set args={--kubelet-insecure-tls} metrics-server metrics-server/metrics-server --namespace kube-system
```

Com isso você realizará a instalação do Metrics-Server em seu cluster de maneira simples.
No comando acima estamos adicionando o repositório do Metrics-Server, atualizando o repositório, e instalando o Metrics-Server no namespace kube-system, com o argumento --kubelet-insecure-tls, que é necessário para que o Metrics-Server funcione corretamente no KinD.

#### Verificando a Instalação do Metrics Server
Após a instalação do Metrics Server, é uma boa prática verificar se ele foi instalado corretamente e está funcionando como esperado. Execute o seguinte comando para obter a lista de pods no namespace kube-system e verificar se o pod do Metrics Server está em execução:

```
kubectl get pods -n kube-system | grep metrics-server
```

#### Obtendo Métricas
Com o Metrics Server em execução, agora você pode começar a coletar métricas de seu cluster. Aqui está um exemplo de como você pode obter métricas de uso de CPU e memória para todos os seus nodes:

```
kubectl top nodes
```

E para obter métricas de uso de CPU e memória para todos os seus pods:

```
kubectl top pods
```

Esses comandos fornecem uma visão rápida da utilização de recursos em seu cluster, o que é crucial para entender e otimizar o desempenho de seus aplicativos.


