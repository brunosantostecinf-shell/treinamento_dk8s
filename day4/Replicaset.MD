## ReplicaSet

Uma coisa é super importante de saber, quando estamos criando um Deployment no Kubernetes, automaticamente estamos criando além do Deployment um ReplicaSet e esse ReplicaSet é quem vai criar os Pods que estão dentro do Deployment.

Confuso, não?

Não, não é e vou te explicar.

Quando criamos um Deployment, o Kubernetes cria um ReplicaSet para criar e fazer o gerenciamento das réplicas dos Pods em nosso cluster. Ele é o responsável por ficar observando os Pods e garantir o número de réplicas que nós definimos no Deployment.

É possível criar um ReplicaSet sem um Deployment, mas não é uma boa prática, pois o ReplicaSet não tem a capacidade de fazer o gerenciamento de versões dos Pods e também não tem a capacidade de fazer o gerenciamento de RollingUpdate dos Pods.

E olha que interessante, quando estamos fazendo a atualização de uma versão de um Pod com o Deployment, o Deployment cria um novo ReplicaSet para fazer o gerenciamento das réplicas dos Pods e quando a atualização termina, o Deployment remove as réplicas do ReplicaSet antigo e deixa apenas as réplicas do ReplicaSet novo.

Mas ele não remove o ReplicaSet antigo, ele deixa ele lá, pois ele pode ser usado para fazer um Rollback da versão do Pod caso algo dê errado. Sim, quando precisamos fazer o Rollback de uma atualização em nossos Pods, o Deployment somente muda o ReplicaSet que está sendo usado para fazer o gerenciamento das réplicas dos Pods, passando a utilizar o ReplicaSet antigo.

Vamos fazer isso na prática?

Acho que vai te ajudar a entender melhor.

