### Instalando o Kyverno

A instalação do Kyverno em um cluster Kubernetes pode ser feita de várias maneiras, incluindo a utilização de um gerenciador de pacotes como o Helm, ou diretamente através de arquivos YAML. Aqui estão os passos básicos para instalar o Kyverno:

Utilizando Helm
O Helm é um gerenciador de pacotes para Kubernetes, que facilita a instalação e gerenciamento de aplicações. Para instalar o Kyverno com Helm, siga estes passos:

Adicione o Repositório do Kyverno:

```
helm repo add kyverno https://kyverno.github.io/kyverno/
helm repo update
```
Instale o Kyverno:

Você pode instalar o Kyverno no namespace kyverno usando o seguinte comando:

```
helm install kyverno kyverno/kyverno --namespace kyverno --create-namespace
```

Verificando a Instalação
Após a instalação, é importante verificar se o Kyverno foi instalado corretamente e está funcionando como esperado.

Verifique os Pods:
```
kubectl get pods -n kyverno
```

Este comando deve mostrar os pods do Kyverno em execução no namespace especificado.

Verifique os CRDs:

```
kubectl get crd | grep kyverno
```

Este comando deve listar os CRDs relacionados ao Kyverno, indicando que foram criados corretamente.

Lembrando que é sempre importante consultar a documentação oficial para obter as instruções mais atualizadas e detalhadas, especialmente se estiver trabalhando com uma configuração específica ou uma versão mais recente do Kyverno ou do Kubernetes.