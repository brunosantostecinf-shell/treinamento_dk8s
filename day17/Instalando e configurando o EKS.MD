### Criando um Cluster EKS com Network Policies

Eu acredito que nessa altura do treinamento, você já saiba o que é um cluster EKS, certo?

Mas mesmo assim, eu vou fazer um pequena apresentação somente para resfrescar a sua memória ou então ajudar quem está chegando agora por aqui. hahaha

O EKS é o Kubernetes gerenciado pela AWS, mas o que isso significa?

Quando falamos sobre clusters Kubernetes gerenciados, estamos falando que que não precisaremos nos preocupar com a instalação e configuração do Kubernetes, pois isso será feito pela AWS. Nós precisaremos apenas criar o nosso cluster e gerenciar as nossas aplicações.

Como você já sabe, nós temos dois tipos de Nodes, os Nodes do Control Plane e os Nodes Workers. No EKS, os Nodes do Control Plane são gerenciados pela AWS, ou seja, não precisaremos nos preocupar com eles. Já os Workers, nós precisaremos criar e gerenciar, pelo menos na maioria dos casos.

Antes de começar, vamos entender os três tipos de cluster EKS que podemos ter:

* Managed Node Groups: Nesse tipo de cluster, os Nodes Workers são gerenciados pela AWS, ou seja, não precisaremos nos preocupar com eles. A AWS irá criar e gerenciar os Nodes Workers para nós. Esse tipo de cluster é ideal para quem não quer se preocupar com a administração dos Nodes Workers.
* Self-Managed Node Groups: Nesse tipo de cluster, os Nodes Workers são gerenciados por nós, ou seja, precisaremos criar e gerenciar os Nodes Workers. Esse tipo de cluster é ideal para quem quer ter o controle total sobre os Nodes Workers.
* Fargate: Nesse tipo de cluster, os Nodes Workers são gerenciados pela AWS, mas não precisaremos nos preocupar com eles, pois a AWS irá criar e gerenciar os Nodes Workers para nós. Esse tipo de cluster é ideal para quem não quer se preocupar com a administração dos Nodes Workers, mas também não quer se preocupar com a criação e gerenciamento dos Nodes Workers.

Evidentemente, cada tipo tem os seus prós e contras, e você precisa analisar o seu cenário para escolher o tipo de cluster que melhor se encaixa nas suas necessidades.

Na maioria dos casos de ambientes produtivos, iremos optar pelo tipo Self-Managed Node Groups, pois assim teremos o controle total sobre os Nodes Workers, podendo customiza-lo e gerencia-lo da forma que acharmos melhor. Agora, se você não quer se preocupar com a administração dos Nodes Workers, você pode optar pelo tipo Managed Node Groups ou Fargate.

Quando optamos pelo Fargate, temos que levar em consideração que não teremos acesso aos Nodes Workers, pois eles são gerenciados pela AWS. Isso significa menos liberdade e recursos, mas também significa menos preocupação e menos trabalho com a administração dos Nodes Workers.

Para o nosso exemplo, vamos escolher do tipo 'Managed Node Groups', pois assim não precisaremos nos preocupar com a administração dos Nodes Workers, mas lembre-se que você pode escolher o tipo que melhor se encaixa nas suas necessidades.

Para criar o cluster vamos utilizar o EKSCTL, que é uma ferramenta de linha de comando que nos ajuda a criar e gerenciar clusters EKS. Você pode encontrar mais informações sobre o EKSCTL aqui.

Ela acabou se tornando uma das formas oficiais de criar e gerenciar clusters EKS, e é a ferramenta que eu mais utilizo para criar e gerenciar clusters EKS. Alias, acredito que seja a ferramenta mais utilizada para criar clusters EKS, quando não estamos utilizando alguma ferramenta de IaC, como o Terraform, por exemplo.

#### Instalando o EKSCTL

Para instalar o EKSCTL, você pode seguir as instruções aqui.

Ele está disponível para Linux, MacOS e Windows. Além de ser possível de rodar em um container Docker.

Em nosso exemplo, vamos utilizar o Linux, claro! hahaha

Para instalar o EKSCTL no Linux, você pode executar o seguinte comando:
```bash
# for ARM systems, set ARCH to: `arm64`, `armv6` or `armv7`
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH

curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"

# (Optional) Verify checksum
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check

tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz

sudo mv /tmp/eksctl /usr/local/bin
```

Eu literalmente copiei e colei o comando acima do site do EKSCTL, então não tem erro. Mas lembrando, sempre é bom consultar o site oficial para verificar se não houve nenhuma alteração.

Aqui estamos fazendo o seguinte:

* Definindo a arquitetura do nosso sistema, no meu caso, amd64. Você pode verificar a arquitetura do seu sistema executando o comando uname -m.
* Definindo a plataforma do nosso sistema, no meu caso, Linux_amd64.
* Baixando o binário do EKSCTL.
* Descompactando o binário do EKSCTL.
* Movendo o binário do EKSCTL para o diretório /usr/local/bin.
* Validando a instalação do EKSCTL:

```bash
eksctl version
```
A saída deve ser algo parecido com isso:

0.169.0
Essa é a versão do EKSCTL que estamos utilizando no momento de criação desse treinamento, mas você pode estar utilizando uma versão mais recente.

#### Instalando o AWS CLI

Agora que temos o EKSCTL instalado, precisamos ter o AWS CLI instalado e configurado, pois o EKSCTL utiliza o AWS CLI para se comunicar com a AWS. Você pode encontrar mais informações sobre o AWS CLI aqui.

O AWS CLI é uma ferramenta de linha de comando que nos ajuda a interagir com os serviços da AWS, ele é super poderoso e é uma das ferramentas mais utilizadas para interagir com os serviços da AWS.

Vou colar abaixo os comandos para instalar o AWS CLI no Linux, mas lembre-se, sempre é bom consultar o site oficial para verificar se não houve nenhuma alteração.

```bash
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```
Validando a instalação do AWS CLI:

```
aws --version
```

No meu caso, a versão que estou utilizando é a seguinte:

```
aws-cli/2.15.10 Python/3.11.6 Linux/6.5.0-14-generic exe/x86_64.zorin.17 prompt/off
```

Agora que temos o AWS CLI instalado, precisamos configurar o AWS CLI, para isso, você pode executar o seguinte comando:

```
aws configure
```

Aqui você precisa informar as suas credenciais da AWS, que você pode encontrar [aqui](https://console.aws.amazon.com/iam/home?#/securi
ty_credentials).

As informações que você precisa informar são:

* AWS Access Key ID
* AWS Secret Access Key
* Default region name
* Default output format

O seu Access Key ID e o Secret Access Key pode ser encontrados aqui. Já a região fica a seu critero, eu vou utilizar a região us-east-1, mas você pode utilizar a região que preferir. E por fim, o formato de saída, eu vou utilizar o formato json, mas você pode utilizar outra opção, como text, por exemplo.

#### Criando o Cluster EKS

Agora que temos o AWS CLI instalado e configurado, podemos criar o nosso cluster EKS.

Podemos cria-lo através da linha de comando somente ou então podemos criar um arquivo de configuração para facilitar a criação do cluster.

Primeiro vou trazer o comando que iremos utilizar e na sequência vou explicar o que estamos fazendo e trazer o arquivo de configuração.

```bash
eksctl create cluster --name=eks-cluster --version=1.28 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
```

Aqui estamos fazendo o seguinte:

* eksctl create cluster: Comando para criar o cluster.
* --name: Nome do cluster.
* --version: Versão do Kubernetes que iremos utilizar, no meu caso, 1.28.
* --region: Região onde o cluster será criado, no meu caso, us-east-1.
* --nodegroup-name: Nome do Node Group.
* --node-type: Tipo de instância que iremos utilizar para os Nodes Workers, no meu caso, t3.medium.
* --nodes: Quantidade de Nodes Workers que iremos criar, no meu caso, 2.
* --nodes-min: Quantidade mínima de Nodes Workers que iremos criar, no meu caso, 1.
* --nodes-max: Quantidade máxima de Nodes Workers que iremos criar, no meu caso, 3.
* --managed: Tipo de Node Group que iremos utilizar, no meu caso, managed.

A saída do comando deve ser algo parecido com isso:

```
2024-01-26 16:12:39 [ℹ]  eksctl version 0.168.0
2024-01-26 16:12:39 [ℹ]  using region us-east-1
2024-01-26 16:12:40 [ℹ]  skipping us-east-1e from selection because it doesn't support the following instance type(s): t3.medium
2024-01-26 16:12:40 [ℹ]  setting availability zones to [us-east-1c us-east-1d]
2024-01-26 16:12:40 [ℹ]  subnets for us-east-1c - public:192.168.0.0/19 private:192.168.64.0/19
2024-01-26 16:12:40 [ℹ]  subnets for us-east-1d - public:192.168.32.0/19 private:192.168.96.0/19
2024-01-26 16:12:40 [ℹ]  nodegroup "eks-cluster-nodegroup" will use "" [AmazonLinux2/1.28]
2024-01-26 16:12:40 [ℹ]  using Kubernetes version 1.28
2024-01-26 16:12:40 [ℹ]  creating EKS cluster "eks-cluster" in "us-east-1" region with managed nodes
2024-01-26 16:12:40 [ℹ]  will create 2 separate CloudFormation stacks for cluster itself and the initial managed nodegroup
2024-01-26 16:12:40 [ℹ]  if you encounter any issues, check CloudFormation console or try 'eksctl utils describe-stacks --region=us-east-1 --cluster=eks-cluster'
2024-01-26 16:12:40 [ℹ]  Kubernetes API endpoint access will use default of {publicAccess=true, privateAccess=false} for cluster "eks-cluster" in "us-east-1"
2024-01-26 16:12:40 [ℹ]  CloudWatch logging will not be enabled for cluster "eks-cluster" in "us-east-1"
2024-01-26 16:12:40 [ℹ]  you can enable it with 'eksctl utils update-cluster-logging --enable-types={SPECIFY-YOUR-LOG-TYPES-HERE (e.g. all)} --region=us-east-1 --cluster=eks-cluster'
2024-01-26 16:12:40 [ℹ]  
2 sequential tasks: { create cluster control plane "eks-cluster", 
    2 sequential sub-tasks: { 
        wait for control plane to become ready,
        create managed nodegroup "eks-cluster-nodegroup",
    } 
}
2024-01-26 16:12:40 [ℹ]  building cluster stack "eksctl-eks-cluster-cluster"
2024-01-26 16:12:40 [ℹ]  deploying stack "eksctl-eks-cluster-cluster"
2024-01-26 16:13:10 [ℹ]  waiting for CloudFormation stack "eksctl-eks-cluster-cluster"
2024-01-26 16:13:41 [ℹ]  waiting for CloudFormation stack "eksctl-eks-cluster-cluster"
2024-01-26 16:14:41 [ℹ]  waiting for CloudFormation stack "eksctl-eks-cluster-cluster"
2024-01-26 16:24:48 [ℹ]  building managed nodegroup stack "eksctl-eks-cluster-nodegroup-eks-cluster-nodegroup"
2024-01-26 16:24:49 [ℹ]  deploying stack "eksctl-eks-cluster-nodegroup-eks-cluster-nodegroup"
2024-01-26 16:24:49 [ℹ]  waiting for CloudFormation stack "eksctl-eks-cluster-nodegroup-eks-cluster-nodegroup"
2024-01-26 16:27:40 [ℹ]  waiting for the control plane to become ready
2024-01-26 16:27:40 [✔]  saved kubeconfig as "/home/jeferson/.kube/config"
2024-01-26 16:27:40 [ℹ]  no tasks
2024-01-26 16:27:40 [✔]  all EKS cluster resources for "eks-cluster" have been created
2024-01-26 16:27:41 [ℹ]  nodegroup "eks-cluster-nodegroup" has 2 node(s)
2024-01-26 16:27:41 [ℹ]  node "ip-192-168-55-232.ec2.internal" is ready
2024-01-26 16:27:41 [ℹ]  node "ip-192-168-7-245.ec2.internal" is ready
2024-01-26 16:27:41 [ℹ]  waiting for at least 1 node(s) to become ready in "eks-cluster-nodegroup"
2024-01-26 16:27:41 [ℹ]  nodegroup "eks-cluster-nodegroup" has 2 node(s)
2024-01-26 16:27:41 [ℹ]  node "ip-192-168-55-232.ec2.internal" is ready
2024-01-26 16:27:41 [ℹ]  node "ip-192-168-7-245.ec2.internal" is ready
2024-01-26 16:27:42 [ℹ]  kubectl command should work with "/home/jeferson/.kube/config", try 'kubectl get nodes'
2024-01-26 16:27:42 [✔]  EKS cluster "eks-cluster" in "us-east-1" region is ready
```

Pronto, cluster deployado com sucesso! 😄

Para vizualizar o nosso cluster, podemos executar o seguinte comando:

```bash
kubectl get nodes
```
A saída deve ser algo parecido com isso:

```bash
ip-192-168-22-217.ec2.internal   Ready    <none>   20m   v1.28.5-eks-5e0fdde
ip-192-168-50-0.ec2.internal     Ready    <none>   20m   v1.28.5-eks-5e0fdde
```
Agora vamos fazer o seguinte, vamos criar arquivo de configuração do EKSCTL para facilitar a criação do cluster da próxima vez. Para isso, vamos criar um arquivo chamado eksctl.yaml e adicionar o seguinte conteúdo:

```yaml
apiVersion: eksctl.io/v1alpha5 # Versão da API do EKSCTL
kind: ClusterConfig # Tipo de recurso que estamos criando

metadata: # Metadados do recurso
  name: eks-cluster # Nome do cluster
  region: us-east-1 # Região onde o cluster será criado

managedNodeGroups: # Node Groups que serão criados, estamos utilizando o tipo Managed Node Groups
- name: eks-cluster-nodegroup # Nome do Node Group
  instanceType: t3.medium # Tipo de instância que iremos utilizar para os Nodes Workers
  desiredCapacity: 2 # Quantidade de Nodes Workers que iremos criar
  minSize: 1 # Quantidade mínima de Nodes Workers que iremos criar
  maxSize: 3 # Quantidade máxima de Nodes Workers que iremos criar
```

Conforme você pode ver, estamos criando um cluster EKS com a mesma configuração que utilizamos anteriormente, mas agora estamos utilizando um arquivo de configuração para facilitar e deixar tudo mais bonitinho. 😄

Para criar o cluster utilizando o arquivo de configuração, você pode executar o seguinte comando:

```
eksctl create cluster -f eksctl.yaml
```

A saída deve ser algo parecido com a que tivemos anteriormente, nada de novo para acrescentar aqui.

Pronto, agora que temos o nosso cluster EKS criado, vamos instalar o CNI da AWS e habilitar o Network Policy nas configurações avançadas do CNI.

#### Instalando o AWS VPC CNI Plugin

O AWS VPC CNI Plugin é um plugin de rede que permite que os Pods se comuniquem com outros Pods e serviços dentro do cluster. Ele também permite que os Pods se comuniquem com serviços fora do cluster, como o Amazon S3, por exemplo.

Vamos utilizar o EKSCTL para instalar o AWS VPC CNI Plugin, para isso, você pode executar o seguinte comando:

```bash
eksctl create addon --name vpc-cni --version v1.16.0-eksbuild.1 --cluster eks-cluster --force
```
Lembrando que você precisa substituir o nome do cluster e a versão do CNI pela versão do seu cluster.

Você pode verificar o link abaixo para verificar a versão do CNI que você precisa utilizar:

https://docs.aws.amazon.com/eks/latest/userguide/managing-vpc-cni.html

Você deve escolher a versão do CNI de acordo com a versão do Kubernetes que você está utilizando, então fique atento a isso.

Bem, voltando ao comando, o que estamos fazendo aqui é o seguinte:

* eksctl create addon: Comando para instalar um Addon no cluster.
* --name: Nome do Addon.
* --version: Versão do Addon.
* --cluster: Nome do cluster.
* --force: Forçar a instalação do Addon.

A saída deve ser algo parecido com isso:

```
2024-01-28 14:12:44 [!]  no IAM OIDC provider associated with cluster, try 'eksctl utils associate-iam-oidc-provider --region=us-east-1 --cluster=eks-cluster'
2024-01-28 14:12:44 [ℹ]  Kubernetes version "1.28" in use by cluster "eks-cluster"
2024-01-28 14:12:44 [!]  OIDC is disabled but policies are required/specified for this addon. Users are responsible for attaching the policies to all nodegroup roles
2024-01-28 14:12:45 [ℹ]  creating addon
2024-01-28 14:13:49 [ℹ]  addon "vpc-cni" active
```

Apesar de ser o CNI padrão do EKS, ele não vem instalado por padrão, então precisamos instala-lo manualmente.

Caso queira visualizar os Addons instalados no seu cluster, você pode executar o seguinte comando:

```
eksctl get addon --cluster eks-cluster
```
A saída deve ser algo parecido com isso:

```
2024-01-28 14:16:44 [ℹ]  Kubernetes version "1.28" in use by cluster "eks-cluster"
2024-01-28 14:16:44 [ℹ]  getting all addons
2024-01-28 14:16:45 [ℹ]  to see issues for an addon run `eksctl get addon --name <addon-name> --cluster <cluster-name>`
NAME	VERSION			STATUS	ISSUES	IAMROLE	UPDATE AVAILABLE	CONFIGURATION VALUES
vpc-cni	v1.16.0-eksbuild.1	ACTIVE	0		v1.16.2-eksbuild.1	
```

Ou então acessar o console da AWS e verificar os Addons instalados no seu cluster, conforme a imagem abaixo:

Alt text

Pronto, CNI instalado com sucesso! 😄

#### Habilitando o Network Policy nas Configurações Avançadas do CNI

Agora que temos o CNI da AWS instalado, precisamos habilitar o Network Policy nas configurações avançadas do CNI, para isso, precisamos acessar o console da AWS e seguir os seguintes passos:

* Acessar o console da AWS.
* Acessar o serviço EKS.
* Selecionar o seu cluster.
* Selecionar a aba Add-ons.
* Selecionar o edit do Addon vpc-cni.
* Configuração Avançada do CNI.
* Habilitar o Network Policy.
Alt text

Depois de alguns minutos, você pode acessar o Addon vpc-cni novamente e verificar se o Network Policy está habilitado e atualizado com o Network Policy habilitado.

Alt text

Pronto, cluster configurado! Agora já podemos continuar com o nosso exemplo. 😄

#### Instalando o Nginx Ingress Controller

Para que tudo funcione bem em nosso exemplo, vamos instalar o Nginx Ingress Controller. É importante observar a versão do Ingress Controller que você está instalando, pois as versões mais recentes ou mais antigas podem não ser compatíveis com o Kubernetes que você está usando. Para este tutorial, vamos usar a versão 1.9.5.
No seu terminal, execute os seguintes comandos:

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.9.5/deploy/static/provider/cloud/deploy.yaml
```
Verifique se o Ingress Controller foi instalado corretamente:
```
kubectl get pods -n ingress-nginx
```
Você pode utilizar a opção wait do kubectl, assim quando os pods estiverem prontos, ele irá liberar o shell, veja:

```
kubectl wait --namespace ingress-nginx \
  --for=condition=ready pod \
  --selector=app.kubernetes.io/component=controller \
  --timeout=90s
```
Pronto, Ingress Controller instalado com sucesso! 😄

Vamos seguir em frente e criar a nossa aplicação de exemplo.

