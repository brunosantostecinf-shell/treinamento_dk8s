### Criando um RoleBinding para o nosso usuário

A RoleBinding é o recurso que associa um usuário a uma Role, ou seja, é através da RoleBinding que definimos qual usuário terá acesso a qual Role, é como se tivessemos um crachá de Developer, a Role seria o crachá, e a RoleBinding seria o crachá com o nome do usuário. Faz sentindo?

Para isso, vamos criar um arquivo chamado developer-rolebinding.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: DeveloperRoleBinding
  namespace: dev
subjects:
- kind: User
  name: developer
roleRef:
  kind: Role
  name: developer
  apiGroup: rbac.authorization.k8s.io
```

No arquivo acima, estamos definindo as seguintes informações:

* apiVersion: Versão da API que estamos utilizando para criar o nosso usuário.
* kind: Tipo do recurso que estamos criando, no caso, um RoleBinding.
* metadata.name: Nome da nossa RoleBinding.
* metadata.namespace: Namespace que a nossa RoleBinding será criada.
* subjects: Usuários que terão acesso a Role.
* subjects.kind: Tipo do usuário, que no caso é User.
* subjects.name: Nome do usuário, que no caso é developer.
* roleRef: Referência da Role que o usuário terá acesso.
* roleRef.kind: Tipo da Role, que no caso é Role.
* roleRef.name: Nome da Role, que no caso é developer.
* roleRef.apiGroup: Grupo de recursos da Role, que no caso é rbac.authorization.k8s.io.

Nada de outro mundo, e mais uma vez está super claro o que iremos ter, que é o usuário developer com a Role developer no namespace dev.

Agora que já temos o nosso arquivo criado, bora aplica-lo:

```
kubectl apply -f developer-rolebinding.yaml
```

Para verificar se a nossa RoleBinding foi criada com sucesso, vamos listar as RoleBindings do cluster:

```
kubectl get rolebindings -n dev
```

A saída:

```
NAME                   ROLE             AGE
DeveloperRoleBinding   Role/developer   9s
```

Para ver detalhes da RoleBinding, vamos utilizar o comando kubectl describe:

```
kubectl describe rolebinding DeveloperRoleBinding -n dev
```

A saída será algo parecido com isso:

```
Name:         DeveloperRoleBinding
Labels:       <none>
Annotations:  <none>
Role:
  Kind:  Role
  Name:  developer
Subjects:
  Kind  Name       Namespace
  ----  ----       ---------
  User  developer  
```

Pronto, RoleBinding criada com sucesso, agora vamos testar o nosso usuário.

#### Adicionando o certificado do usuário no kubeconfig

Agora que já temos o nosso usuário criado, precisamos adicionar o certificado do usuário no kubeconfig, para que possamos acessar o cluster com o nosso usuário.

Para isso, vamos utilizar o comando kubectl config set-credentials:

```
kubectl config set-credentials developer --client-certificate=developer.crt --client-key=developer.key --embed-certs=true
```

O comando kubectl config set-credentials é utilizado para adicionar um novo usuário no kubeconfig, e ele recebe os seguintes parametros:

* --client-certificate: Caminho do certificado do usuário.
* --client-key: Caminho da chave do usuário.
* --embed-certs: Indica se o certificado deve ser embutido no kubeconfig.

No nosso caso, estamos passando o caminho do certificado e da chave do usuário, e estamos indicando que o certificado deve ser embutido no kubeconfig.

Agora precisamos criar um contexto para o nosso usuário, para isso, vamos utilizar o comando kubectl config set-context:

```
kubectl config set-context developer --cluster=NOME-DO-CLUSTER --namespace=dev --user=developer
```

Caso você não se lembre o que é um contexto no Kubernetes, eu vou te ajudar a relembrar. Um contexto é um conjunto de configurações que definem o acesso a um cluster, ou seja, um contexto é composto por um cluster, um usuário e um namespace. Quando você cria um novo usuário, você precisa criar um novo contexto para ele, para que ele possa acessar o cluster.

Para que você possa pegar os nomes do cluster, basta utilizar o comando kubectl config get-clusters, assim você poderá pegar o nome do cluster que você quer utilizar.

Com isso, o nosso novo usuário está pronto para ser utilizado, mas antes vamos verificar se ele está funcionando.

