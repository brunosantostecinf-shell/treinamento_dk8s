# Utilizando Tokens para Service Accounts

Uma das formas de autentição no Kubernetes é através de Tokens, que são utilizados para autenticar Service Accounts, que são contas de serviço utilizadas, normalmente, por aplicações que rodam no cluster ou serviços que precisam acessar o cluster.

Os Tokens são gerados automaticamente pelo Kubernetes, e são utilizados para autenticar Service Accounts, e eles são armazenados em Secrets, que são recursos do Kubernetes que armazenam informações sensíveis, como por exemplo, chaves privadas, senhas, tokens, etc.

O primeiro passo para a criação de um Service Account utilizando Tokens é criar o Service Account, então bora começar os trablahos!

## Criando um Service Account

Podemos criar service accounts utilizando o comando kubectl ou através de um arquivo YAML, e é isso que vamos fazer, criar um arquivo chamado service-account.yaml com o seguinte conteúdo:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: service-account-example
  namespace: default
```

Nada de novo acima, certo? No arquivo estamos definindo o seguinte:

- `apiVersion`: Versão da API que estamos utilizando para criar o nosso Service Account.
- `kind`: Tipo do recurso que estamos criando, no caso, um Service Account.
- `metadata.name`: Nome do nosso Service Account.
- `metadata.namespace`: Namespace que o nosso Service Account será criado.

Agora vamos aplicar o arquivo no cluster:

```bash
kubectl apply -f service-account.yaml
```

Caso queira criar utilizando a linha de comando, basta utilizar o comando `kubectl create serviceaccount NOME-DO-SERVICE-ACCOUNT`.

Agora vamos ver os detalhes da nossa Service Account:

```bash
kubectl get serviceaccounts service-account-example -o yaml
```

A saída será algo parecido com isso:

```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"service-account-example","namespace":"default"}}
  creationTimestamp: "2024-02-05T10:20:19Z"
  name: service-account-example
  namespace: default
  resourceVersion: "2472"
  uid: 0d21af3f-e242-477b-8557-983533562274
```

Pronto, Service Account criado com sucesso!

Agora vamos criar o Secret que irá armazenar o Token do Service Account.

## Criando um Secret para o Service Account

Novamente aqui podemos utilizar o comando kubectl ou um arquivo YAML, e é isso que vamos fazer, criar um arquivo chamado service-account-secret.yaml com o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: service-account-example-token
  annotations:
    kubernetes.io/service-account.name: service-account-example
type: kubernetes.io/service-account-token
```

Aqui estamos falando o seguinte para o Kubernetes:

- `apiVersion`: Versão da API que estamos utilizando para criar o nosso Secret.
- `kind`: Tipo do recurso que estamos criando, no caso, um Secret.
- `metadata.name`: Nome do nosso Secret.
- `metadata.annotations`: Anotações do nosso Secret.
- `metadata.annotations.kubernetes.io/service-account.name`: Nome do nosso Service Account.
- `type`: Tipo do Secret, que no caso é kubernetes.io/service-account-token.

Pronto, hora de aplicar o arquivo:

```bash
kubectl apply -f service-account-secret.yaml
```

Vamos ver se está tudo certo com o nosso Secret:

```bash
kubectl get secrets service-account-example-token -o yaml
```

A saída será algo parecido com isso:

```
Name:         service-account-example-token
Namespace:    default
Labels:       <none>
Annotations:  kubernetes.io/service-account.name: service-account-example
              kubernetes.io/service-account.uid: 0d21af3f-e242-477b-8557-983533562274

Type:  kubernetes.io/service-account-token

Data
====
ca.crt:     1099 bytes
namespace:  7 bytes
token:      eyJhbGciOiJSUzI1NiIsImtpZCI6IkhicVNQcnRtbVBOa0tzVm00WUJfZjMwajNMZVN1bl9peXo0WVpUUHVjSTQifQ.eyJpc3MiOiJrdWJlcm5ldGVzL3NlcnZpY2VhY2NvdW50Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9uYW1lc3BhY2UiOiJkZWZhdWx0Iiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZWNyZXQubmFtZSI6InNlcnZpY2UtYWNjb3VudC1leGFtcGxlLXRva2VuIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQubmFtZSI6InNlcnZpY2UtYWNjb3VudC1leGFtcGxlIiwia3ViZXJuZXRlcy5pby9zZXJ2aWNlYWNjb3VudC9zZXJ2aWNlLWFjY291bnQudWlkIjoiMGQyMWFmM2YtZTI0Mi00NzdiLTg1NTctOTgzNTMzNTYyMjc0Iiwic3ViIjoic3lzdGVtOnNlcnZpY2VhY2NvdW50OmRlZmF1bHQ6c2VydmljZS1hY2NvdW50LWV4YW1wbGUifQ.B_N9F9gTucLFryhyTMNJxjCoEAa1q5NmbtUJ95RUNwEUUKY3KapDxezrFMZCA7VG-I7YbukRdYKusYmtWnZLvlgm2UHw1CY4dmzlqBwu3-XhOms8OHVAu1JVbGujgjljPEQy8jPFrkYRz356L_5FUzka9Noj3i2pj5R1wOrJz1O2KAw99RhKI1ctKtXnk4O0VJJ4fdmDUf0Ox0PQt4oNgGr28ZRIpyZkB0RSjhO3Wb6QgsGPoTBKHYnRo3zcCNymEkF8iJBGyyv4WyMx5zr4prCUpeSHqg-2m6vI_KZ9pu0pCJVJ1wV25vU6WOsyY56UR87a7jeBh83m-sOM8SAqOA
```

Já era! Secret está lá e com o Token do Service Account.

Nós não precisamos gerar o Token, isso foi feito automaticamente pelo Kubernetes, pois quando criamos a nossa Secret, nós informamos que a Secret é do tipo kubernetes.io/service-account-token, e também informamos o nome do nosso Service Account. Com isso o Kubernetes gerou automaticamente o Token para o nosso Service Account especificado e armazenou na Secret. Simples como voar!

Agora, para acessar o nosso Token, uma forma que eu gosto bastante é utilizando o comando kubectl get secret, e com o comando `kubectl get secret NOME-DO-SECRET -o jsonpath='{.data.token}' | base64 --decode`, assim você consegue pegar o Token do Service Account.

Olha o comando completo:

```bash
kubectl get secret service-account-example-token -o jsonpath='{.data.token}' | base64 --decode
```

Dessa forma você consegue pegar o Token do Service Account, já decodificado, e utilizar para autenticar a sua aplicação ou serviço no cluster.

## Criando uma Role para o Service Account

Agora precisamos criar uma Role para o nosso Service Account.

Aqui não temos nada de novo, pois já vimos como criar Roles, então vamos criar um arquivo chamado service-account-role.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  name: service-account-role
  namespace: default
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list", "watch"]
```

Com a definição da Role acima, estamos falando que quem utilizar essa Role terá permissão para get, list e watch nos recursos pods do namespace default, somente!

Vamos aplicar o arquivo:

```bash
kubectl apply -f service-account-role.yaml
```

## Criando um RoleBinding para o Service Account

Agora precisamos criar um RoleBinding para o nosso Service Account.

Vamos criar um arquivo chamado service-account-rolebinding.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: service-account-rolebinding
  namespace: default
subjects:
- kind: ServiceAccount
  name: service-account-example
  namespace: default
roleRef:
  kind: Role
  name: service-account-role
  apiGroup: rbac.authorization.k8s.io
```

Aqui estamos conectando a nossa Role com o nosso Service Account, e estamos dizendo que o nosso Service Account terá todas as permissões definidas na Role service-account-role no namespace default.

Vamos aplicar o arquivo:

```bash
kubectl apply -f service-account-rolebinding.yaml
```

Vamos listar as Roles e RoleBindings do namespace default:

```bash
kubectl get roles
kubectl get rolebindings
```

Estão lá, mais uma etapa concluída!

Agora que temos tudo o que precisamos, a nossa Service Account está pronta para ser utilizada.

## Utilizando o Token do Service Account

Para o nossa exemplo, vamos criar um Pod que irá utilizar o Token do Service Account para acessar o cluster.

Vamos criar um arquivo chamado pod-service-account.yaml com o seguinte conteúdo:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-service-account
  namespace: default
spec:
  serviceAccountName: service-account-example
  containers:
  - name: curl-container
    image: curlimages/curl
    command: ["sleep", "infinity"]
```

A informação nova que temos nessa definição de Pod é o campo `serviceAccountName`, que é onde informamos o nome do nosso Service Account.

Vamos aplicar o arquivo:

```bash
kubectl apply -f pod-service-account.yaml
```

Agora vamos listar os Pods do namespace default:

```
NAME                  READY   STATUS    RESTARTS   AGE
pod-service-account   1/1     Running   0          6s
```

Os Tokens são montados dentro do container do Pod no caminho `/var/run/secrets/kubernetes.io/serviceaccount`, e o Token do Service Account está no arquivo token, vamos confirmar isso:

```bash
kubectl exec -it pod-service-account -- ls /var/run/secrets/kubernetes.io/serviceaccount
```

A saída será algo parecido com isso:

```
ca.crt     namespace  token
```

Onde:

- `ca.crt`: É o certificado do cluster.
- `namespace`: É o namespace do Pod.
- `token`: É o Token do Service Account.

Vamos dar uma olhada no conteúdo do arquivo token:

```bash
kubectl exec -it pod-service-account -- cat /var/run/secrets/kubernetes.io/serviceaccount/token
```

Agora temos o nosso Token:

```
eyJhbGciOiJSUzI1NiIsImtpZCI6IkhicVNQcnRtbVBOa0tzVm00WUJfZjMwajNMZVN1bl9peXo0WVpUUHVjSTQifQ.eyJhdWQiOlsiaHR0cHM6Ly9rdWJlcm5ldGVzLmRlZmF1bHQuc3ZjLmNsdXN0ZXIubG9jYWwiXSwiZXhwIjoxNzM4NjY1OTY3LCJpYXQiOjE3MDcxMjk5NjcsImlzcyI6Imh0dHBzOi8va3ViZXJuZXRlcy5kZWZhdWx0LnN2Yy5jbHVzdGVyLmxvY2FsIiwia3ViZXJuZXRlcy5pbyI6eyJuYW1lc3BhY2UiOiJkZWZhdWx0IiwicG9kIjp7Im5hbWUiOiJwb2Qtc2VydmljZS1hY2NvdW50IiwidWlkIjoiY2MyNmQ1MTUtZjYwMS00MTU3LWIzNDAtZDUzM2Q0OTdhZjE5In0sInNlcnZpY2VhY2NvdW50Ijp7Im5hbWUiOiJzZXJ2aWNlLWFjY291bnQtZXhhbXBsZSIsInVpZCI6IjBkMjFhZjNmLWUyNDItNDc3Yi04NTU3LTk4MzUzMzU2MjI3NCJ9LCJ3YXJuYWZ0ZXIiOjE3MDcxMzM1NzR9LCJuYmYiOjE3MDcxMjk5NjcsInN1YiI6InN5c3RlbTpzZXJ2aWNlYWNjb3VudDpkZWZhdWx0OnNlcnZpY2UtYWNjb3VudC1leGFtcGxlIn0.O-XrTx9b1ZqvFpedKH6lBsQeHeP2qn3bEuL1WVKjCgMhd2TtjKAJOgDJH2Spye-7BQF1bwa2KyAwoWCauDDZ4I0XqwjVRSUF8dUahFmbAL9fMzrieEA2oPkNuPnwCCF57Csls52J-1z9GdIElgG-Z48aksrl0iIB1l7y5TBKS3Za7W2XDBdl1sFx1fKsAlz6k_NPaC49I21NJlmk9i3LxwctvDn6oxMjbCjBVk4fFVnK3y1HsdR_3trCLKH-FDgy59jD96Ume4VPgQMfmXNigObyJ6t3qcL53IRnqdY0xrFNOwHjwIMxBqQSTGSf42xUBWVjfKHZ_gMTX_Va-Nyk9A
```

Agora vamos fazer um bom teste utilizando o nosso Token.

## Testando o Token com a API do Kubernetes

No Kubernetes é possível listar os recursos do cluster utilizando a API do Kubernetes, e para isso podemos utilizar o comando curl, mas antes vamos entender como funciona a autenticação com o Token.

Para autenticar com o Token, precisamos enviar o Token no header Authorization da requisição, e o valor do header Authorization deve ser Bearer seguido do Token.

Vamos entender o endereco da API do Kubernetes, que é `https://kubernetes.default.svc`, e o recurso que queremos listar, que é pods, e o namespace que queremos listar, que é default, então a URL da nossa requisição será:

```
https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

Se eu quiser listar todos os Services do namespace default, a URL será:

```
https://kubernetes.default.svc/api/v1/namespaces/default/services
```

Se for em um namespace específico, basta trocar o default pelo nome do namespace.

Agora vamos fazer a requisição utilizando o curl, mas antes precisamos entrar no container do Pod, então vamos fazer isso:

```bash
kubectl exec -it pod-service-account -- sh
```

Agora vamos fazer a requisição utilizando o curl:

```bash
curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/pods
```

Com isso estamos utilizando o curl para fazer a requisição para a API do Kubernetes, passando o Token do Service Account no header Authorization, e estamos listando os Pods do namespace default.

Eu não vou colar a saída aqui, pois ela é gigante, então faça o teste e veja os detalhes dos Pods do namespace default.

Se você tentar listar os Pods de outro namespace, você não terá permissão, pois a Role que criamos para o nosso Service Account é somente para o namespace default, e a mesma coisa para os outros recursos do cluster como Services, Deployments, etc.

Vamos tentar listar os Services do namespace default:

```bash
curl -k -H "Authorization: Bearer $(cat /var/run/secrets/kubernetes.io/serviceaccount/token)" https://kubernetes.default.svc/api/v1/namespaces/default/services
```

Recebemos um erro, pois o nosso Service Account não tem permissão para listar os Services do namespace default.

```json
{
  "kind": "Status",
  "apiVersion": "v1",
  "metadata": {},
  "status": "Failure",
  "message": "services is forbidden: User \"system:serviceaccount:default:service-account-example\" cannot list resource \"services\" in API group \"\" in the namespace \"default\"",
  "reason": "Forbidden",
  "details": {
    "kind": "services"
  },
  "code": 403
}
```

Pronto, com isso o nosso teste está completo, e vimos como utilizar o Token do Service Account para autenticar na API do Kubernetes.

Agora é só praticar e ficar uma pessoa expert no assunto!

## Removendo o Service Account

Para remover o Service Account é super simples, basta remover o Service Account, o Secret e a RoleBinding relacionado ao Service Account.

```bash
kubectl delete serviceaccount NOME-DO-SERVICE-ACCOUNT
kubectl delete secret NOME-DO-SECRET
kubectl delete rolebinding NOME-DO-ROLEBINDING
```

Pronto, cluster limpo!
