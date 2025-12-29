O que é um StatefulSet?
Os StatefulSets são uma funcionalidade do Kubernetes que gerencia o deployment e o scaling de um conjunto de Pods, fornecendo garantias sobre a ordem de deployment e a singularidade desses Pods.

Diferente dos Deployments e Replicasets que são considerados stateless (sem estado), os StatefulSets são utilizados quando você precisa de mais garantias sobre o deployment e scaling. Eles garantem que os nomes e endereços dos Pods sejam consistentes e estáveis ao longo do tempo.

 

Quando usar StatefulSets?
Os StatefulSets são úteis para aplicações que necessitam de um ou mais dos seguintes:

Identidade de rede estável e única.
Armazenamento persistente estável.
Ordem de deployment e scaling garantida.
Ordem de rolling updates e rollbacks garantida.
Algumas aplicações que se encaixam nesses requisitos são bancos de dados, sistemas de filas e quaisquer aplicativos que necessitam de persistência de dados ou identidade de rede estável.
E como ele funciona?
Os StatefulSets funcionam criando uma série de Pods replicados. Cada réplica é uma instância da mesma aplicação que é criada a partir do mesmo spec, mas pode ser diferenciada por seu índice e hostname.

Ao contrário dos Deployments e Replicasets, onde as réplicas são intercambiáveis, cada Pod em um StatefulSet tem um índice persistente e um hostname que se vinculam a sua identidade.

Por exemplo, se um StatefulSet tiver um nome giropops e um spec com três réplicas, ele criará três Pods: giropops-0, giropops-1, giropops-2. A ordem dos índices é garantida. O Pod giropops-1 não será iniciado até que o Pod giropops-0 esteja disponível e pronto.

A mesma garantia de ordem é aplicada ao scaling e aos updates.

O StatefulSet e os volumes persistentes
Um aspecto chave dos StatefulSets é a integração com Volumes Persistentes. Quando um Pod é recriado, ele se reconecta ao mesmo Volume Persistente, garantindo a persistência dos dados entre as recriações dos Pods.

Por padrão, o Kubernetes cria um PersistentVolume para cada Pod em um StatefulSet, que é então vinculado a esse Pod para a vida útil do StatefulSet.

Isso é útil para aplicações que precisam de um armazenamento persistente e estável, como bancos de dados.

O StatefulSet e o Headless Service
Para entender a relação entre o StatefulSet e o Headless Service, é preciso primeiro entender o que é um Headless Service.

No Kubernetes, um serviço é uma abstração que define um conjunto lógico de Pods e uma maneira de acessá-los. Normalmente, um serviço tem um IP e encaminha o tráfego para os Pods. No entanto, um Headless Service é um tipo especial de serviço que não tem um IP próprio. Em vez disso, ele retorna diretamente os IPs dos Pods que estão associados a ele.

Agora, o que isso tem a ver com os StatefulSets?

Os StatefulSets e os Headless Services geralmente trabalham juntos no gerenciamento de aplicações stateful. O Headless Service é responsável por permitir a comunicação de rede entre os Pods em um StatefulSet, enquanto o ` gerencia o deployment e o scaling desses Pods.

Aqui está como eles funcionam juntos:

Quando um StatefulSet é criado, ele geralmente é associado a um Headless Service. Ele é usado para controlar o domínio DNS dos Pods criados pelo StatefulSet. Cada Pod obtém um nome de host DNS que segue o formato: <pod-name>.<service-name>.<namespace>.svc.cluster.local. Isso permite que cada Pod seja alcançado individualmente.

Por exemplo, se você tiver um StatefulSet chamado giropops com três réplicas e um Headless Service chamado nginx, os Pods criados serão giropops-0, giropops-1, giropops-2 e eles terão os seguintes endereços de host DNS: giropops-0.nginx.default.svc.cluster.local, giropops-1.nginx.default.svc.cluster.local, giropops-2.nginx.default.svc.cluster.local.

Essa combinação de StatefulSets com Headless Services permite que aplicações stateful, como bancos de dados, tenham uma identidade de rede estável e previsível, facilitando a comunicação entre diferentes instâncias da mesma aplicação.

Bem, agora que você já entendeu como isso funciona, eu acho que já podemos dar os primeiros passos com o StatefulSet.

