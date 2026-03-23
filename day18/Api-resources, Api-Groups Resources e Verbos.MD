### Criando um Role para o nosso usuário

Quando criamos um novo Usuário ou ServiceAccount no Kubernetes, ele não tem acesso a nada no cluster, para que ele possa acessar os recursos do cluster, precisamos criar um Role e associar ele ao usuário.

A definição da Role consiste em um arquivo onde definimos quais são as permissões que o usuário terá no cluster, e para quais recursos ele terá acesso. Dentro da Role é onde definimos:

* Qual é o namespace que o usuário terá acesso.
* Quais apiGroups o usuário terá acesso.
* Quais recursos o usuário terá acesso.
* Quais verbos o usuário terá acesso.

#### apiGroups

São os grupos de recursos do Kubernetes, que são divididos em core e named, você pode consultar todos os grupos de recursos do Kubernetes através do comando kubectl api-resources.

Vamos listar os grupos de recursos do Kubernetes:
```
kubectl api-resources
```

A lista é longa, mas o resultado será algo parecido com isso:

```
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
bindings                                       v1                                     true         Binding
componentstatuses                 cs           v1                                     false        ComponentStatus
configmaps                        cm           v1                                     true         ConfigMap
endpoints                         ep           v1                                     true         Endpoints
events                            ev           v1                                     true         Event
limitranges                       limits       v1                                     true         LimitRange
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim
persistentvolumes                 pv           v1                                     false        PersistentVolume
pods                              po           v1                                     true         Pod
podtemplates                                   v1                                     true         PodTemplate
replicationcontrollers            rc           v1                                     true         ReplicationController
resourcequotas                    quota        v1                                     true         ResourceQuota
secrets                                        v1                                     true         Secret
serviceaccounts                   sa           v1                                     true         ServiceAccount
services                          svc          v1                                     true         Service
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
controllerrevisions                            apps/v1                                true         ControllerRevision
daemonsets                        ds           apps/v1                                true         DaemonSet
deployments                       deploy       apps/v1                                true         Deployment
replicasets                       rs           apps/v1                                true         ReplicaSet
statefulsets                      sts          apps/v1                                true         StatefulSet
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler
cronjobs                          cj           batch/v1                               true         CronJob
jobs                                           batch/v1                               true         Job
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
leases                                         coordination.k8s.io/v1                 true         Lease
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice
events                            ev           events.k8s.io/v1                       true         Event
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
ingresses                         ing          networking.k8s.io/v1                   true         Ingress
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding
roles                                          rbac.authorization.k8s.io/v1           true         Role
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

Onde a primeira coluna é o nome do recurso, a segunda coluna é o nome abreviado do recurso, a terceira coluna é a versão da API que o recurso está, a quarta coluna indica se o recurso é ou não Namespaced, e a quinta coluna é o tipo do recurso.

Vamos dar uma olhada em um recurso específico, por exemplo, o recurso pods:

```
kubectl api-resources | grep pods
```

O resultado será algo parecido com isso:

```
NAME       SHORTNAMES   APIVERSION     NAMESPACED   KIND
pods       po           v1             true         Pod
Onde:
```

* NAME: Nome do recurso.
* SHORTNAMES: Nome abreviado do recurso.
* APIVERSION: Versão da API que o recurso está.
* NAMESPACED: Indica se o recurso é ou não Namespaced.
* KIND: Tipo do recurso.

Mas o que é um recurso Namespaced? Um recurso Namespaced é um recurso que pode ser criado dentro de um namespace, por exemplo, um Pod, um Deployment, um Service, etc. Já um recurso que não é Namespaced, é um recurso que não pode ser criado dentro de um namespace, por exemplo, um Node, um PersistentVolume, um ClusterRole, etc. Fácil né?

Agora, como eu sei qual é o apiGroup de um recurso? Bem, o apiGroup de um recurso é o nome do grupo de recursos que ele pertence, por exemplo, o recurso pods pertence ao grupo de recursos core, e o recurso deployments pertence ao grupo de recursos apps. Quando o recurso é do tipo core ele não precisa ser especificado no apiGroup, pois o Kubernetes já entende que ele pertence ao grupo de recursos core, esse é o famoso apiVersion: v1.

Ja o apiVersion: apps/v1 indica que o recurso pertence ao grupo de recursos apps, e a versão da API é a v1. No apps temos recursos importantes como o deployments, replicasets, daemonsets, statefulsets, etc.

#### Recursos

São os recursos do Kubernetes, que são divididos em core e named, você pode consultar todos os recursos do Kubernetes através do comando kubectl api-resources.

Os recursos chamados de core são os recursos que já vem instalados no Kubernetes, e os recursos chamados de named são os recursos que são instalados através de Custom Resource Definitions, ou CRD's, como por exemplo o ServiceMonitor do Prometheus.

Vamos listar os recursos do Kubernetes:

```
kubectl api-resources --namespaced=false
```

O resultado será algo parecido com isso:

```
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND
componentstatuses                 cs           v1                                     false        ComponentStatus
namespaces                        ns           v1                                     false        Namespace
nodes                             no           v1                                     false        Node
persistentvolumes                 pv           v1                                     false        PersistentVolume
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition
apiservices                                    apiregistration.k8s.io/v1              false        APIService
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver
csinodes                                       storage.k8s.io/v1                      false        CSINode
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment
```

Assim podemos saber quais são os recursos que são nativos do Kubernetes, e quais são os recursos que são instalados através de CRD's, que são os Custom Resources Definitions.

Então o nome do recurso é o nome que utilizamos para criar o recurso, por exemplo, pods, deployments, services, etc.

#### Verbos

Os verbos definem o que o usuário pode fazer com o recurso, por exemplo, o usuário pode criar, listar, atualizar, deletar, etc.

Para que você possa visualizar os verbos que podem ser utilizados, vamos utilizar o comando kubectl api-resources com o parametro -o wide:

```
kubectl api-resources -o wide
```

O resultado será algo parecido com isso:

```
NAME                              SHORTNAMES   APIVERSION                             NAMESPACED   KIND                             VERBS                                                        CATEGORIES
bindings                                       v1                                     true         Binding                          create                                                       
componentstatuses                 cs           v1                                     false        ComponentStatus                  get,list                                                     
configmaps                        cm           v1                                     true         ConfigMap                        create,delete,deletecollection,get,list,patch,update,watch   
endpoints                         ep           v1                                     true         Endpoints                        create,delete,deletecollection,get,list,patch,update,watch   
events                            ev           v1                                     true         Event                            create,delete,deletecollection,get,list,patch,update,watch   
limitranges                       limits       v1                                     true         LimitRange                       create,delete,deletecollection,get,list,patch,update,watch   
namespaces                        ns           v1                                     false        Namespace                        create,delete,get,list,patch,update,watch                    
nodes                             no           v1                                     false        Node                             create,delete,deletecollection,get,list,patch,update,watch   
persistentvolumeclaims            pvc          v1                                     true         PersistentVolumeClaim            create,delete,deletecollection,get,list,patch,update,watch   
persistentvolumes                 pv           v1                                     false        PersistentVolume                 create,delete,deletecollection,get,list,patch,update,watch   
pods                              po           v1                                     true         Pod                              create,delete,deletecollection,get,list,patch,update,watch   all
podtemplates                                   v1                                     true         PodTemplate                      create,delete,deletecollection,get,list,patch,update,watch   
replicationcontrollers            rc           v1                                     true         ReplicationController            create,delete,deletecollection,get,list,patch,update,watch   all
resourcequotas                    quota        v1                                     true         ResourceQuota                    create,delete,deletecollection,get,list,patch,update,watch   
secrets                                        v1                                     true         Secret                           create,delete,deletecollection,get,list,patch,update,watch   
serviceaccounts                   sa           v1                                     true         ServiceAccount                   create,delete,deletecollection,get,list,patch,update,watch   
services                          svc          v1                                     true         Service                          create,delete,deletecollection,get,list,patch,update,watch   all
mutatingwebhookconfigurations                  admissionregistration.k8s.io/v1        false        MutatingWebhookConfiguration     create,delete,deletecollection,get,list,patch,update,watch   api-extensions
validatingwebhookconfigurations                admissionregistration.k8s.io/v1        false        ValidatingWebhookConfiguration   create,delete,deletecollection,get,list,patch,update,watch   api-extensions
customresourcedefinitions         crd,crds     apiextensions.k8s.io/v1                false        CustomResourceDefinition         create,delete,deletecollection,get,list,patch,update,watch   api-extensions
apiservices                                    apiregistration.k8s.io/v1              false        APIService                       create,delete,deletecollection,get,list,patch,update,watch   api-extensions
controllerrevisions                            apps/v1                                true         ControllerRevision               create,delete,deletecollection,get,list,patch,update,watch   
daemonsets                        ds           apps/v1                                true         DaemonSet                        create,delete,deletecollection,get,list,patch,update,watch   all
deployments                       deploy       apps/v1                                true         Deployment                       create,delete,deletecollection,get,list,patch,update,watch   all
replicasets                       rs           apps/v1                                true         ReplicaSet                       create,delete,deletecollection,get,list,patch,update,watch   all
statefulsets                      sts          apps/v1                                true         StatefulSet                      create,delete,deletecollection,get,list,patch,update,watch   all
tokenreviews                                   authentication.k8s.io/v1               false        TokenReview                      create                                                       
localsubjectaccessreviews                      authorization.k8s.io/v1                true         LocalSubjectAccessReview         create                                                       
selfsubjectaccessreviews                       authorization.k8s.io/v1                false        SelfSubjectAccessReview          create                                                       
selfsubjectrulesreviews                        authorization.k8s.io/v1                false        SelfSubjectRulesReview           create                                                       
subjectaccessreviews                           authorization.k8s.io/v1                false        SubjectAccessReview              create                                                       
horizontalpodautoscalers          hpa          autoscaling/v2                         true         HorizontalPodAutoscaler          create,delete,deletecollection,get,list,patch,update,watch   all
cronjobs                          cj           batch/v1                               true         CronJob                          create,delete,deletecollection,get,list,patch,update,watch   all
jobs                                           batch/v1                               true         Job                              create,delete,deletecollection,get,list,patch,update,watch   all
certificatesigningrequests        csr          certificates.k8s.io/v1                 false        CertificateSigningRequest        create,delete,deletecollection,get,list,patch,update,watch   
leases                                         coordination.k8s.io/v1                 true         Lease                            create,delete,deletecollection,get,list,patch,update,watch   
endpointslices                                 discovery.k8s.io/v1                    true         EndpointSlice                    create,delete,deletecollection,get,list,patch,update,watch   
events                            ev           events.k8s.io/v1                       true         Event                            create,delete,deletecollection,get,list,patch,update,watch   
flowschemas                                    flowcontrol.apiserver.k8s.io/v1beta3   false        FlowSchema                       create,delete,deletecollection,get,list,patch,update,watch   
prioritylevelconfigurations                    flowcontrol.apiserver.k8s.io/v1beta3   false        PriorityLevelConfiguration       create,delete,deletecollection,get,list,patch,update,watch   
ingressclasses                                 networking.k8s.io/v1                   false        IngressClass                     create,delete,deletecollection,get,list,patch,update,watch   
ingresses                         ing          networking.k8s.io/v1                   true         Ingress                          create,delete,deletecollection,get,list,patch,update,watch   
networkpolicies                   netpol       networking.k8s.io/v1                   true         NetworkPolicy                    create,delete,deletecollection,get,list,patch,update,watch   
runtimeclasses                                 node.k8s.io/v1                         false        RuntimeClass                     create,delete,deletecollection,get,list,patch,update,watch   
poddisruptionbudgets              pdb          policy/v1                              true         PodDisruptionBudget              create,delete,deletecollection,get,list,patch,update,watch   
clusterrolebindings                            rbac.authorization.k8s.io/v1           false        ClusterRoleBinding               create,delete,deletecollection,get,list,patch,update,watch   
clusterroles                                   rbac.authorization.k8s.io/v1           false        ClusterRole                      create,delete,deletecollection,get,list,patch,update,watch   
rolebindings                                   rbac.authorization.k8s.io/v1           true         RoleBinding                      create,delete,deletecollection,get,list,patch,update,watch   
roles                                          rbac.authorization.k8s.io/v1           true         Role                             create,delete,deletecollection,get,list,patch,update,watch   
priorityclasses                   pc           scheduling.k8s.io/v1                   false        PriorityClass                    create,delete,deletecollection,get,list,patch,update,watch   
csidrivers                                     storage.k8s.io/v1                      false        CSIDriver                        create,delete,deletecollection,get,list,patch,update,watch   
csinodes                                       storage.k8s.io/v1                      false        CSINode                          create,delete,deletecollection,get,list,patch,update,watch   
csistoragecapacities                           storage.k8s.io/v1                      true         CSIStorageCapacity               create,delete,deletecollection,get,list,patch,update,watch   
storageclasses                    sc           storage.k8s.io/v1                      false        StorageClass                     create,delete,deletecollection,get,list,patch,update,watch   
volumeattachments                              storage.k8s.io/v1                      false        VolumeAttachment                 create,delete,deletecollection,get,list,patch,update,watch  
```

Perceba que agora temos uma nova coluna, a coluna VERBS, onde temos todos os verbos que podem ser utilizados com o recurso, e a coluna CATEGORIES, onde temos a categoria do recurso. Mas o nosso foco aqui é nos verbos, então vamos dar uma olhada neles.

Os verbos são divididos em:

* create: Permite que o usuário crie um recurso.
* delete: Permite que o usuário delete um recurso.
* deletecollection: Permite que o usuário delete uma coleção de recursos.
* get: Permite que o usuário obtenha um recurso.
* list: Permite que o usuário liste os recursos.
* patch: Permite que o usuário atualize um recurso.
* update: Permite que o usuário atualize um recurso.
* watch: Permite que o usuário acompanhe as alterações de um recurso.

Com isso, vamos pegar exemplo da linha do recurso pods:

```
NAME   SHORTNAMES   APIVERSION     NAMESPACED   KIND     VERBS                     CATEGORIES
pods   po           v1             true         Pod      create,delete,deletecollection,get,list,patch,update,watch   all
```

Com isso sabemos que o usuário pode criar, deletar, deletar uma coleção, obter, listar, atualizar e acompanhar as alterações de um Pod. Simplão demais, hein!

#### Criando a Role

Agora que já sabemos muito sobre os resources, apiGroups e verbs, vamos criar a nossa Role para o nosso usuário.

Para isso, vamos criar um arquivo chamado developer-role.yaml com o seguinte conteúdo:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
 name: developer
 namespace: dev
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list", "update", "create", "delete"]
```

No arquivo acima, estamos definindo as seguintes informações:

* apiVersion: Versão da API que estamos utilizando para criar o nosso usuário.
* kind: Tipo do recurso que estamos criando, no caso, uma Role.
* metadata.name: Nome da nossa Role.
* metadata.namespace: Namespace que a nossa Role será criada.
* rules: Regras da nossa Role.
* rules.apiGroups: Grupos de recursos que a nossa Role terá acesso.
* rules.resources: Recursos que a nossa Role terá acesso.
* rules.verbs: Verbos que a nossa Role terá acesso.

Eu tenho certeza que está fácil para você fazer a leitura da Role, que é basicamente o que o nosso usuário pode fazer no cluster, mas em resumo o que estamos falando é que o usuário que estiver usando essa Role, poderá fazer tudo com o recurso pods no namespace dev. Simples como voar!

Lembre-se que essa Role pode ser reutilizada por outros usuários, e que você pode criar quantas Roles quiser, e que você pode criar Roles para outros perfis de usuários e para outros recursos, como por exemplo, deployments, services, configmaps, etc.

Ahhh, antes de mais nada, vamos criar o namespace dev:

```
kubectl create ns dev
```

Agora que já temos o nosso arquivo e a namespace criados, vamos aplicar ele no cluster:

```
kubectl apply -f developer-role.yaml
```

Para verificar se a nossa Role foi criada com sucesso, vamos listar as Roles do cluster:

```
kubectl get roles -n dev
```

A saída:

```
NAME        CREATED AT
developer   2024-01-31T11:32:08Z
```

Para ver os detalhes da Role, vamos utilizar o comando kubectl describe:

```
kubectl describe role developer -n dev
```

A saída será algo parecido com isso:

```
Name:         developer
Labels:       <none>
Annotations:  <none>
PolicyRule:
  Resources  Non-Resource URLs  Resource Names  Verbs
  ---------  -----------------  --------------  -----
  pods       []                 []              [get watch list update create delete]
```

Pronto, a nossa Role já está criada, porém ainda não associamos ela ao nosso usuário, para isso, vamos criar um RoleBinding.

