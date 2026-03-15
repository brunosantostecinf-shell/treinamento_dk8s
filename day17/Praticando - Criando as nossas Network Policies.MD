### Criando Regras de Network Policy

#### Ingress

Em nosso exemplo, tanto a nossa aplicação quanto o Redis estão rodando no mesmo namespace, o namespace giropops. Por padrão, os Pods podem se comunicar livremente entre si. Vamos criar uma Network Policy para restringir o acesso ao Redis, permitindo que somente que pods do namespace giropops possam acessar o Redis.

Para isso, vamos criar o arquivo permitir-redis-somente-mesmo-ns.yaml com o seguinte conteúdo:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: redis-allow-same-namespace
  namespace: giropops
spec:
  podSelector:
    matchLabels:
      app: redis
  ingress:
  - from:
    - podSelector: {}
```

Vamos entender o que estamos fazendo aqui:

* apiVersion: Versão da API que estamos utilizando.
* kind: Tipo de recurso que estamos criando.
* metadata: Metadados do recurso.
    * name: Nome da Network Policy.
    * namespace: Namespace onde a Network Policy será criada.
* spec: Especificação da Network Policy.
    * podSelector: Seletor de Pods que serão afetados pela Network Policy.
        * matchLabels: Labels dos Pods que serão afetados pela Network Policy.
    * ingress: Regras de entrada.
        * from: Origem do tráfego.
            * podSelector: Seletor de Pods que podem acessar os Pods selecionados, nesse caso, todos os Pods do namespace giropops.

Sempre que tiver o {} significa que estamos selecionando todos os Pods que atendem aos critérios especificados, nesse caso, todos os Pods do namespace giropops pois não estamos especificando nenhum critério.

Vamos aplicar a nossa Network Policy:

```
kubectl apply -f permitir-redis-somente-mesmo-ns.yaml -n giropops
```

Vamos testar o acesso ao Redis a partir de um Pod fora do namespace giropops, para isso vamos user o comando redis ping:

```
kubectl run -it --rm --image redis redis-client -- redis-cli -h redis-service.giropops.svc.cluster.local ping
```

Se tudo estiver funcionando corretamente, você não receberá nenhuma resposta, pois o acesso ao Redis está bloqueado para Pods fora do namespace giropops.

Agora, se você executar o mesmo comando, porém de dentro da Namespace giropops, você deverá receber a mensagem PONG, pois o acesso ao Redis está permitido para Pods dentro do namespace giropops! 😄

Vamos testar:

```
kubectl run -it --rm -n giropops --image redis redis-client -- redis-cli -h redis-service.giropops.svc.cluster.local ping
```

A saída deve ser algo parecido com isso:

```
If you don't see a command prompt, try pressing enter.

PONG
Session ended, resume using 'kubectl attach redis-client -c redis-client -i -t' command when the pod is running
pod "redis-client" deleted
```

Pronto, agora que temos a nossa Network Policy funcionando!

Agora vamos queremos bloquear todo o acesso de entrada para os Pods do namespace giropops, para isso, vamos criar o arquivo nao-permitir-nada-externo.yaml com o seguinte conteúdo:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-ns
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
```

O que mudou aqui foi o seguinte:

* policyTypes: Tipo de política que estamos criando, nesse caso, estamos criando uma política de entrada.
* ingress: Regras de entrada.
    * from: Origem do tráfego.
        * namespaceSelector: Seletor de Namespaces que podem acessar os Pods selecionados, nesse caso, somente o namespace giropops.
        * matchLabels: Labels dos Namespaces que podem acessar os Pods selecionados, nesse caso, somente o namespace giropops.
        * kubernetes.io/metadata.name: Nome do Namespace.
        * giropops: Valor do Nome do Namespace.

Simples, assim, estamos bloqueando todo o tráfego de entrada para os Pods do namespace giropops, menos para os Pods do próprio namespace giropops.

Vamos aplicar a nossa Network Policy:

```
kubectl apply -f nao-permitir-nada-externo.yaml -n giropops
```

Vamos testar o acesso ao Redis a partir de um Pod fora do namespace giropops, para isso vamos user o comando redis ping:

```
kubectl run -it --rm --image redis redis-client -- redis-cli -h redis-service.giropops.svc.cluster.local ping
```

Nada de novo, certo, porém vamos testar o acesso a nossa aplicação a partir de um Pod fora do namespace giropops, para isso vamos user o comando curl:

```
kubectl run -it --rm --image curlimages/curl curl-client -- curl giropops-senhas.giropops.svc
```

Se tudo estiver funcionando corretamente, você não receberá nenhuma resposta, pois o acesso a nossa aplicação está bloqueado para Pods fora do namespace giropops.

Agora, se você executar o mesmo comando, porém de dentro da Namespace giropops, você deverá receber a mensagem Giropops Senhas, pois o acesso a nossa aplicação está permitido para Pods dentro do namespace giropops! 😄

Vamos testar:

```
kubectl run -it --rm -n giropops --image curlimages/curl curl-client -- curl giropops-senhas.giropops.svc
```

Tudo funcionando maravilhosamente bem! De dentro do mesmo namespace, podemos acessar a nossa aplicação e o Redis, mas de fora do namespace, não podemos acessar nada! 😄

Mas com isso temos um problema, pois o nosso Ingress Controller não consegue acessar a nossa aplicação, pois ele está fora do namespace giropops, então vamos criar uma Network Policy para permitir o acesso ao nosso Ingress Controller.

Para isso, vamos criar o arquivo permitir-ingress-controller.yaml com o seguinte conteúdo:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-ns-and-ingress-controller
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
```

Aqui a solução foi super simples, pois somente adicionamos mais um seletor de Namespaces, para permitir o acesso ao nosso Ingress Controller. Com isso, tudo que estiver dentro do namespace ingress-nginx e giropops poderá acessar os Pods do namespace giropops.

Aqui poderiamos ter uma melhoria no código utilizando o matchExpressions, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-same-ns-and-ingress-controller
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchExpressions:
        - key: kubernetes.io/metadata.name
          operator: In
          values: ["ingress-nginx", "giropops"]
```

O resultado seria o mesmo, porém o código ficaria mais limpo e mais fácil de entender.

Agora pode testar o acesso a nossa aplicação a partir de um Pod fora do namespace giropops, para isso vamos user o comando curl:

kubectl run -it --rm --image curlimages/curl curl-client -- curl giropops-senhas.giropops.svc
Aqui você não conseguirá acessar a nossa aplicação, pois o acesso a nossa aplicação está bloqueado para Pods fora do namespace giropops, agora se você executar o mesmo comando, porém de dentro da Namespace giropops, tudo funcionará bem!

Porém, sempre que você utilizar o endereço do Ingress da nossa aplicação, você conseguirá normalmente, pois liberamos o acesso ao nosso Ingress Controller, portanto os clientes da nossa app que acessarão via internet, conseguirão acessar normalmente, porém Pods fora do namespace giropops não conseguirão acessar a nossa aplicação. Lindo não é mesmo? 😄

Somente uma observação importante:

```
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
```

Uma coisa que deve ser bem entendido no momento de criar as suas regras são os operadores lógicos, pois eles podem mudar completamente o resultado da sua Network Policy. No nosso exemplo, estamos utilizando o operador lógico OR, ou seja, estamos permitindo o acesso ao nosso Ingress Controller OU ao nosso namespace giropops.

Se você quiser permitir o acesso ao nosso Ingress Controller E ao nosso namespace giropops, você deve utilizar o operador lógico AND, veja:

```
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
```

Nesse caso a regra funcionará da seguinte forma, somente os Pods que estiverem dentro do namespace ingress-nginx E do namespace giropops poderão acessar os Pods do namespace giropops, ou seja, teriamos problemas.

Teste aí e veja o que acontece. 😄

Nós podemos ter uma abortagem diferente, podemos ter um regra que irá bloquear todo o tráfego de entrada, e depois criar regras para permitir o tráfego de entrada para os Pods que precisam, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: default-deny-all
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  - Egress
```
Agora estamos bloqueando todo o tráfego de entrada para os Pods do namespace giropops, pois estamos utilizando o ingress: [] para bloquear todo o tráfego de entrada. Mais uma vez, estamos usando o [] vazio para selecionar todos os Pods e bloquear todo o tráfego de entrada, pois não estamos especificando nenhum critério.

O policyTypes é um campo obrigatório, e nele você deve especificar o tipo de política que você está criando, nesse caso, estamos criando uma política de entrada e saída, por isso estamos utilizando o Ingress e o Egress.

Vamos aplicar:

```
kubectl apply -f deny-all-ingress.yaml -n giropops
```

Agora vamos criar uma regra para que a nossa App consiga acessar o Redis, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-redis-app-only
  namespace: giropops
spec:
  podSelector:
    matchLabels:
      app: redis
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: giropops-senhas
      namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
    ports:
    - protocol: TCP
      port: 6379
```

Com isso temos mais um regra para permitir o acesso ao Redis e a App somente entre eles e somente nas portas 6379 e 5000.

Vamos aplicar:

```
kubectl apply -f permitir-somente-ingress-entre-app-redis-mesmo-ns.yaml -n giropops
```

Pronto, fizemos mais uma camada de segurança, agora somente a nossa aplicação pode acessar o Redis, e somente nas portas 6379 e 5000, mas ainda temos um problema, pois o nosso Ingress Controller não consegue acessar a nossa aplicação, e com isso, nossos clientes não irão conseguir acessar a nossa aplicação, então vamos criar uma Network Policy para permitir o acesso ao nosso Ingress Controller, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ingress-controller
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: ingress-nginx
    ports:
    - protocol: TCP
      port: 5000
```

Pronto, agora o nosso Ingress Controller consegue acessar a nossa aplicação, e com isso, nossos clientes também conseguem acessar a nossa aplicação!

Mas ainda temos um problema, os nossos Pods não conseguem acessar o DNS do cluster, então vamos criar uma Network Policy para permitir o acesso ao DNS do cluster e com isso o Pod de nossa App conseguirá acessar o Redis tranquilamente

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-dns-access
  namespace: giropops
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: kube-system
      podSelector:
        matchLabels:
          k8s-app: kube-dns
    ports:
    - protocol: UDP
      port: 53
```

Pronto, um problema a menos! 😄
Mas ainda temos outro!

Quando criamos a regra de Egress bloqueando tudo, bloqueamos também o tráfego de saída de todos os Pods do Namespace giropops, e com isso, o nosso Pod de App não consegue acessar o Redis.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-ns
  namespace: giropops
spec:
  podSelector:
    matchLabels: {}
  policyTypes:
  - Egress
  egress:
  - to:
    - namespaceSelector:
        matchLabels:
          kubernetes.io/metadata.name: giropops
      podSelector:
        matchLabels:
          app: redis
```

Pronto, agora acredito que todos os problemas foram resolvidos e podemos acessar a nossa App e o Redis normalmente! 😄

Outra opção bem interessante de utilizar é o ipBlock, com ele você pode especificar um endereço IP ou um CIDR para permitir o acesso, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ip-block
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress:
  - from:
    - ipBlock:
        cidr: 172.18.0.0/16
```

Com a regra acima estamos falando que nós queremos permitir o acesso somente para o range de IPs dentro do CIDR 172.18.0.0/16, ou seja, somente os Pods que estiverem dentro desse range de IPs poderão acessar os Pods do namespace giropops.

Ainda podemos adionar uma regra de exceção, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: ip-block
  namespace: giropops
spec:
  policyTypes:
  - Ingress
  podSelector: {}
  ingress:
  - from:
    - ipBlock:
        cidr: 172.18.0.0/16
        except:
        - 172.18.0.112/32
```

Com a regra acima, toda a rede 172.18.0.0/16 terá acesso, menos o IP 172.18.0.112 que não terá acesso aos Pods do namespace giropops.

Nós criamos um monte de Network Policies, mas não nos concentramos em entender como ver se elas estão criadas e seus detalhes, então vamos ver como podemos fazer isso.

Para visualizar as Network Policies que estão criadas no seu cluster, você pode executar o seguinte comando:

```
kubectl get networkpolicies -n giropops
```

Para ver os detalhes de uma Network Policy, você pode executar o seguinte comando:

```
kubectl describe networkpolicy <nome-da-network-policy> -n giropops
```

Para deletar uma Network Policy, você pode executar o seguinte comando:

```
kubectl delete networkpolicy <nome-da-network-policy> -n giropops
```

Simples como voar, não?

#### Egress

Nós falamos muito como criar regras de Ingres, ou seja, regras de entrada, mas e as regras de saída? Como podemos criar regras de saída?

Para isso temos o egress, que é muito parecido com o ingress, mas com algumas diferenças, veja:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress
  namespace: giropops
spec:
  podSelector: {}
  policyTypes:
  - Egrees
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
```

Com a regra acima, estamos liberando o acesso para os Pods que atendem aos critérios especificados, nesse caso, somente os Pods que tiverem o label app: redis poderão acessar os Pods do namespace giropops na porta 6379. Com isso, todos os Pods da namespace giropops poderão acessar os Pods que tiverem o label app: redis na porta 6379.

Agora, se quisermos que somente a nossa aplicação possa acessar o Redis, podemos fazer o seguinte:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-egress-only-app
  namespace: giropops
spec:
  policyTypes:
  - Egrees
  podSelector: 
    matchLabels:
      app: giropops-senhas
  egress:
  - to:
    - podSelector:
        matchLabels:
          app: redis
    ports:
    - protocol: TCP
      port: 6379
```

Com a regra acima, somente a nossa aplicação poderá acessar o Redis, pois estamos utilizando o podSelector para selecionar somente os Pods que tiverem o label app: giropops-senhas, ou seja, somente a nossa aplicação poderá acessar o Redis.

Simples demais!
