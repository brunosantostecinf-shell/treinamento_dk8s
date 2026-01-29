Instalando o nosso cluster Kubernetes
Como dissemos, para esse nosso exemplo iremos utilizar o cluster de Kubernetes da AWS, o EKS. Para instalar o nosso cluster EKS, iremos utilizar a ferramenta eksctl, portanto precisamos instalá-la em nossa máquina. Para instalar a ferramenta, basta executar o seguinte comando:

```
ARCH=amd64
PLATFORM=$(uname -s)_$ARCH
curl -sLO "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_$PLATFORM.tar.gz"
curl -sL "https://github.com/eksctl-io/eksctl/releases/latest/download/eksctl_checksums.txt" | grep $PLATFORM | sha256sum --check
tar -xzf eksctl_$PLATFORM.tar.gz -C /tmp && rm eksctl_$PLATFORM.tar.gz
sudo install -m 0755 /tmp/eksctl /usr/local/bin && rm /tmp/eksctl
eksctl verison
```

Precisamos ter o CLI da aws instalado em nossa máquina, para isso, basta executar o seguinte comando:

```
curl "https://awscli.amazonaws.com/awscli-exe-linux-x86_64.zip" -o "awscliv2.zip"
unzip awscliv2.zip
sudo ./aws/install
```

Pronto, agora você já tem o eksctl e o aws instalados em sua máquina.

Para que possamos criar tudo o que precisamos na AWS, é importante que você tenha uma conta na AWS, e que tenha as credenciais de acesso configuradas em sua máquina. Para configurar as credenciais de acesso, basta executar o seguinte comando:

```
aws configure
```

O comando acima irá solicitar que você informe a sua AWS Access Key ID, a sua AWS Secret Access Key, a sua Default region name, e o seu Default output format. Para saber mais sobre como configurar as credenciais de acesso, basta acessar a documentação oficial da AWS.

No comando acima estamos baixando o binário do eksctl compactado e descompactando ele na pasta /tmp, e depois movendo o binário para a pasta /usr/local/bin.

Lembrando que estou instando em uma máquina Linux, caso que esteja utilizando uma máquina Mac ou Windows, basta acessar a página de releases do projeto e baixar a versão adequada para o seu sistema operacional.

E enquanto você faz a instalação, vale a pena mencionar que o eksctl é uma ferramenta criada pela WeaveWorks, empresa que criou o Flux, que é um projeto de GitOps para Kubernetes, além de ter o Weavenet, que é um CNI para Kubernetes, e o Weave Scope, que é uma ferramenta de visualização de clusters de Kubernetes e muito mais, recomendo que vocês dêem uma olhada nos projetos, é sensacional!

Bem, agora você já tem o eksctl instalado em sua máquina, então vamos criar o nosso cluster EKS! Para isso, basta executar o seguinte comando:

```
eksctl create cluster --name=eks-cluster --version=1.24 --region=us-east-1 --nodegroup-name=eks-cluster-nodegroup --node-type=t3.medium --nodes=2 --nodes-min=1 --nodes-max=3 --managed
```

O comando acima irá criar um cluster EKS com o nome eks-cluster, na região us-east-1, com 2 nós do tipo t3.medium, e com um mínimo de 1 nó e um máximo de 3 nós. Além disso, o comando acima irá criar um nodegroup chamado eks-cluster-nodegroup. O eksctl irá cuidar de toda a infraestrutura necessária para o funcionamento do nosso cluster EKS. A versão do Kubernetes que será instalada no nosso cluster será a 1.24.

Após a criação do nosso cluster EKS, precisamos instalar o kubectl em nossa máquina. Para instalar o kubectl, basta executar o seguinte comando:

```
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

O comando acima irá baixar o binário do kubectl e o colocar na pasta /usr/local/bin, e dar permissão de execução para o binário.

Agora que já temos o kubectl instalado em nossa máquina, precisamos configurar o kubectl para utilizar o nosso cluster EKS. Para isso, basta executar o seguinte comando:

```
aws eks --region us-east-1 update-kubeconfig --name eks-cluster
```

Aonde us-east-1 é a região do nosso cluster EKS, e eks-cluster é o nome do nosso cluster EKS. Esse comando é necessário para que o kubectl saiba qual cluster ele deve utilizar, ele irá pegar as credenciais do nosso cluster EKS e armazenar no arquivo ~/.kube/config.

LEMBRE-SE: Você não precisa ter o Kubernetes rodando no EKS, fique a vontade para escolher onde preferir para seguir o treinamento.

Vamos ver se o kubectl está funcionando corretamente? Para isso, basta executar o seguinte comando:

```
kubectl get nodes
```

Se tudo estiver funcionando corretamente, você deverá ver uma lista com os nós do seu cluster EKS. 😄

Antes de seguirmos em frente, vamos conhecer algums comandos do eksctl, para que possamos gerenciar o nosso cluster EKS. Para listar os clusters EKS que temos em nossa conta, basta executar o seguinte comando:

```
eksctl get cluster -A
```

O parametro -A é para listar os clusters EKS de todas as regiões. Para listar os clusters EKS de uma região específica, basta executar o seguinte comando:

```
eksctl get cluster -r us-east-1
```

Para aumentar o número de nós do nosso cluster EKS, basta executar o seguinte comando:

```
eksctl scale nodegroup --cluster=eks-cluster --nodes=3 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
```

Para diminuir o número de nós do nosso cluster EKS, basta executar o seguinte comando:

```
eksctl scale nodegroup --cluster=eks-cluster --nodes=1 --nodes-min=1 --nodes-max=3 --name=eks-cluster-nodegroup -r us-east-1
```

Para deletar o nosso cluster EKS, basta executar o seguinte comando:

```
eksctl delete cluster --name=eks-cluster -r us-east-1
```

Mas não delete o nosso cluster EKS, vamos utilizar ele para os próximos passos! hahahah