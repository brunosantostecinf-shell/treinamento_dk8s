## As Probes do Kubernetes

Antes de seguir, eu queria trazer algo novo além dos dois novos objetos que você já aprendeu no dia de hoje. Eu queria que você saisse do dia de hoje com a segurança que você e capaz de criar um Pod, um Deployment, um ReplicaSet ou um DaemonSet, mas também com a segurança que você pode monitorar o seus suas aplicações que estão rodando dentro do cluster de maneira efetiva e utilizando recursos que o Kubernetes já nos disponibiliza.

O que são as Probes?

As probes são uma forma de você monitorar o seu Pod e saber se ele está em um estado saudável ou não. Com elas é possível assegurar que seus Pods estão rodando e respondendo de maneira correta, e mais do que isso, que o Kubernetes está testando o que está sendo executado dentro do seu Pod.

Hoje nós temos disponíveis três tipos de probes, a livenessProbe, a **readinessProbe** e a **startupProbe**. Vamos ver no detalhe cada uma delas.