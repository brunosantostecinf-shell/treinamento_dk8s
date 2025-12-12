## 🧩 Desafio: O Mistéerio do Pod Faminto

## 📝 Contexto

Você é o Engenheiro DevOps responsável por implantar uma nova aplicação de processamento de dados chamada StressApp. Os desenvolvedores avisaram que essa aplicação pode ser um pouco… “gulosa” no consumo de memória RAM.

Seu objetivo é implantar essa aplicação no cluster Kubernetes, garantindo que ela não derrube o node nem afete outros serviços. Para isso, você irá trabalhar com Pods, Resource Requests/Limits e Namespaces.

## 🚀 Missão

### Parte 1 – Preparando o Terreno

Crie um namespace chamado treinamento-ch2.
Toda a atividade deve acontecer dentro dele para manter o cluster organizado.

### Parte 2 – O Teste de Estresse (O Erro Esperado)

Crie um manifesto YAML para um Pod com as seguintes especificações:

Name: pod-faminto
Image: polinux/stress
Namespace: treinamento-ch2
Command:
stress --vm 1 --vm-bytes 250M --vm-hang 1

Esse comando faz o container tentar alocar 250MB de RAM.

Resources:
Requests:
memory: "100Mi"
Limits:
memory: "200Mi"

Após criar o manifesto, aplique no cluster.

## 📌 Observação:
Use os comandos abaixo para acompanhar o comportamento:

- kubectl get pod -n treinamento-ch2 -w
- kubectl describe pod pod-faminto -n treinamento-ch2


Você deve observar o Pod entrando em falha com o motivo OOMKilled.

### Parte 3 – A Correção

Crie um novo manifesto (ou edite o anterior) para o Pod chamado pod-comportado:
Mantenha a mesma imagem e comando:

- stress --vm 1 --vm-bytes 250M --vm-hang 1

Ajuste os limits de memória para suportar os 250MB solicitados pela aplicação — lembre-se de incluir uma margem de segurança.

Após aplicar o manifesto, o Pod deve ficar com status Running.

## 🧠 Perguntas para Reflexão

Quando o Pod foi morto na Parte 2, qual mecanismo do Linux (visto no Capítulo 1) foi acionado pelo Kubernetes/container runtime para encerrar o processo?

- Qual a diferença prática entre definir um request de 100Mi e um limit de 200Mi?

- Qual valor o Scheduler usa para decidir onde colocar o Pod?

LeSmbre-se: containers são apenas processos isolados.
Se um processo usa mais memória do que o cgroup permite, o kernel assume o controle!
