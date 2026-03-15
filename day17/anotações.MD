```bash
# Exibe o ConfigMap do CoreDNS no namespace kube-system em formato YAML
# e filtra apenas linhas que contenham a palavra "cluster"
# Útil para validar o domínio de cluster configurado, como "cluster.local"
kubectl -n kube-system get configmap coredns -o yaml | grep cluster


# Envia uma requisição HTTP apenas para buscar os headers (-I)
# para o serviço giropops-senhas na porta 5000 usando o DNS interno do cluster
# Útil para validar:
# - resolução DNS interna
# - conectividade com o serviço
# - resposta HTTP da aplicação
curl -I giropops-senhas.giropops.svc.cluster.local:5000


# Tenta fazer uma requisição HTTP para o serviço Redis na porta 6379
# Na prática, Redis não fala HTTP, então o esperado aqui é erro,
# timeout ou resposta inválida para HTTP
# Esse teste serve mais para checar conectividade básica do que o protocolo Redis em si
curl -I redis-service.giropops.svc.cluster.local:6379


# Usa o cliente Redis para conectar no serviço redis-service
# e executar o comando PING
# Se estiver tudo certo, a resposta esperada é:
# PONG
# Este é o teste correto para validar comunicação com Redis
redis-cli -h redis-service.giropops.svc.cluster.local ping


# Aplica no cluster o arquivo YAML da primeira NetworkPolicy
# Cria ou atualiza a policy definida no arquivo
kubectl apply -f day17/netpol/primeira-network-policy.yaml 


# Consulta a NetworkPolicy chamada primeira-netpol
# no namespace giropops
kubectl get netpol -n giropops primeira-netpol 


# Exibe a mesma NetworkPolicy em YAML
# Bom para validar como ela ficou persistida no cluster
kubectl get netpol -n giropops primeira-netpol -o yaml


# Mostra uma descrição detalhada da NetworkPolicy
# incluindo:
# - pods selecionados
# - regras de ingress/egress
# - origens/destinos permitidos
kubectl describe netpol -n giropops primeira-netpol


# Cria um Pod temporário interativo com Alpine e abre um shell
# OBS: o parâmetro correto é --image=alpine
# e não --image: alpine
# Como esse Pod será criado no namespace default,
# ele é útil para testar se a NetworkPolicy permite acesso
# vindo de OUTRO namespace
kubectl run -ti alpine --image=alpine -- sh

# Dentro do Pod Alpine, testa acesso ao Redis via DNS interno
# Se a policy bloquear acesso entre namespaces,
# este teste provavelmente vai falhar
redis-cli -h redis-service.giropops.svc.cluster.local ping


# Cria outro Pod temporário interativo com Alpine,
# mas agora dentro do namespace giropops
# Esse teste é útil para validar acesso a partir do MESMO namespace
kubectl run -ti alpine --image=alpine -n giropops -- sh

# Dentro do Pod Alpine no namespace giropops,
# testa acesso ao Redis
# Se a policy permitir acesso do mesmo namespace,
# a resposta esperada é PONG
redis-cli -h redis-service.giropops.svc.cluster.local ping


# Lista todos os namespaces mostrando também suas labels
# Muito útil para validar namespaceSelector em NetworkPolicies
kubectl get ns --show-labels

# Lista todos os nodes mostrando suas labels
# Muito útil para estudar nodeSelector, nodeAffinity
# e distribuição de workloads
kubectl get nodes --show-labels
```
