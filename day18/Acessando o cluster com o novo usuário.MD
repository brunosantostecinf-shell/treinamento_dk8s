# Acessando o cluster com o novo usuário

Uma vez que o nosso usuário está criado, e que o certificado do usuário está no kubeconfig e que já temos um contexto para o usuário, podemos testar o acesso ao cluster.

Antes de mais nada, vamos listar todos os contextos do kubeconfig:

```
kubectl config get-contexts
```

O nosso novo contexto deve aparecer na lista, e deve ser algo parecido com isso:

```
CURRENT   NAME                CLUSTER        AUTHINFO       NAMESPACE
*         developer           kind-strigus   developer      dev
          kind-strigus        kind-strigus   kind-strigus   
```

Então bora o utilizar o nosso novo contexto:

```
kubectl config use-context developer
```

Pronto, nesse momento estamos utilizando o nosso novo usuário através do nosso novo contexto, agora vamos testar o acesso ao cluster:

```
kubectl get pods
```

Tudo funcionando, certo?

Perceba que o Namespace que estamos utilizando é o dev, onde ele pode fazer tudo com o recurso pods, e que ele não pode fazer nada com os outros recursos, como por exemplo, deployments, services, configmaps, etc. Isso porque a nossa Role está limitada ao recurso pods, e ao namespace dev.

Vamos testar o acesso a um recurso que ele não tem permissão:

```
kubectl get deployments
```

O resultado será algo parecido com isso:

```
Error from server (Forbidden): deployments.apps is forbidden: User "developer" cannot list resource "deployments" in API group "apps" in the namespace "dev"
```

Tudo funcionando perfeitamente! Pra finalizar, vamos criar um Pod simples do Nginx, ele deve ser criado com sucesso, pois o nosso usuário tem permissão para criar Pods no namespace dev:

```
kubectl run nginx --image=nginx -n dev
```

Criado!

Vamos listar os Pods do namespace dev:

```
kubectl get pods
```

E veremos o nosso Pod do Nginx criado:

```
NAME    READY   STATUS    RESTARTS   AGE
nginx   1/1     Running   0          5s
```

Pronto! O nosso usuário está criado e funcionando perfeitamente!