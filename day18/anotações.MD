# Documentação dos comandos

- `openssl genrsa -out developer.key 2048`
  - Gera uma chave RSA privada.
  - `-out developer.key`: grava a chave no arquivo `developer.key`.
  - `2048`: tamanho da chave em bits (recomendado para segurança mínima).

- `openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"`
  - Cria um CSR (Certificate Signing Request) usando a chave privada.
  - `-new`: indica que é uma nova solicitação.
  - `-key developer.key`: usa a chave privada existente `developer.key`.
  - `-out developer.csr`: grava o CSR no arquivo `developer.csr`.
  - `-subj "/CN=developer"`: define o Subject com Common Name `developer` (pode ser FQDN ou identidade adequada).

- `cat developer.csr | base64 | tr -d '\n'`
  - Converte o CSR para base64 em uma linha única (sem quebras de linha).
  - Saída usada em YAML/JSON ou campos de API que exigem CSR inline.

  Exemplo de saída (trecho):
  `LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K...`

- `kubectl get csr`
  - Lista CSRs no cluster e mostra condição (`Pending`, `Approved`, `Denied`, `Issued`).
  - Exemplo de saída:
    - `developer ... Pending`
    - depois de aprovar: `developer ... Approved,Issued`

- `kubectl certificate approve developer`
  - Aprova a CSR chamada `developer`.
  - O controller de certificates gera o certificado final quando aprovado.

- `kubectl get csr developer -o jsonpath='{.status.certificate}' | base64 -d > developer.crt`
  - Extrai o certificado assinado do objeto CSR e grava em `developer.crt`.

- `kubectl get csr developer -o yaml`
  - Verifica se o campo `status.certificate` foi preenchido e se `status.conditions` tem `Approved`.

- `kubectl api-resources`
  - Lista todos os recursos de API disponíveis no cluster (namespaced e cluster-scoped).
  - Mostra: NAME, SHORTNAMES, APIVERSION, NAMESPACED, KIND.
  - Útil para descobrir recursos suportados e suas abreviações.

- `kubectl api-resources --namespaced=false`
  - Lista apenas recursos cluster-scoped (não namespaced).
  - `--namespaced=false`: filtra para recursos que não pertencem a namespaces (ex: nodes, pv, csr).
  - Exemplo: nodes, persistentvolumes, certificatesigningrequests.

- `kubectl api-resources -o wide | grep pods`
  - Lista recursos de API em formato wide (mais colunas) e filtra por "pods".
  - `-o wide`: inclui colunas extras como VERBS (operações suportadas).
  - `| grep pods`: filtra linhas que contenham "pods" (ex: pods, podtemplates, etc.).
  - Útil para verificar detalhes de recursos relacionados a pods.

  Exemplo de saída:
  `pods                              po           v1                                true         Pod                              create,delete,deletecollection,get,list,patch,update,watch   all`

- `kubectl create ns dev`
  - Cria um novo namespace chamado `dev`.
  - Namespaces isolam recursos no cluster (pods, services, etc.).
  - Equivalente a `kubectl create namespace dev`.

- `kubectl apply -f role-developer.yaml`
  - Aplica o manifesto YAML `role-developer.yaml` no cluster.
  - Cria ou atualiza a Role `developer` no namespace `dev` com permissões para pods (get, list, create).

- `kubectl get role -n dev`
  - Lista Roles no namespace `dev`.
  - Mostra: NAME, AGE.
  - `-n dev`: especifica o namespace.

- `kubectl get role -n dev -o yaml`
  - Lista Roles em formato YAML completo.
  - Mostra detalhes como rules, apiGroups, resources, verbs.

- `kubectl describe role -n dev developer`
  - Descreve a Role `developer` no namespace `dev`.
  - Mostra: Name, Namespace, Labels, Annotations, PolicyRule (apiGroups, resources, verbs).

- `kubectl apply -f rolebinding-developer.yaml`
  - Aplica o manifesto YAML `rolebinding-developer.yaml` no cluster.
  - Cria ou atualiza um RoleBinding que vincula a Role `developer` a um usuário ou grupo.

- `kubectl get rolebindings.rbac.authorization.k8s.io -n dev`
  - Lista RoleBindings no namespace `dev`.
  - Mostra: NAME, ROLE, AGE.
  - `rolebindings.rbac.authorization.k8s.io`: nome completo da API (pode usar `rolebinding` ou `rb` como shortname).

- `kubectl get rolebindings.rbac.authorization.k8s.io -n dev -o yaml`
  - Lista RoleBindings em formato YAML completo.
  - Mostra detalhes como roleRef (Role vinculada), subjects (usuários/grupos), namespace.

- `kubectl describe rolebindings.rbac.authorization.k8s.io -n dev`
  - Descreve todos os RoleBindings no namespace `dev`.
  - Mostra: Name, Namespace, Role, Subjects (usuários, grupos, service accounts), Labels, Annotations.

- `kubectl config use-context kind-k8s-control-plane`
  - Alterna para o contexto `kind-k8s-control-plane`.
  - Um contexto define cluster + user + namespace para operações.
  - Comando seguinte opera com este contexto ativo.

- `kubectl config set-credentials developer --client-certificate developer.crt --client-key developer.key --embed-cert`
  - Adiciona credenciais `developer` com certificado e chave privada.
  - `--client-key developer.key`: caminho da chave privada.
  - `--embed-cert`: codifica o certificado inline no kubeconfig (base64).
  - Alternativa: sem `--embed-cert`, usa caminho de arquivo.

- `kubectl config get-users`
  - Lista todos os usuários configurados no kubeconfig.
  - Mostra: NAME (identidades disponíveis).
  - Exemplo: developer, kind-k8s-control-plane.

- `kubectl config get-contexts`
  - Lista todos os contextos configurados no kubeconfig.
  - Mostra: CURRENT, NAME, CLUSTER, AUTHINFO, NAMESPACE.
  - `*` marca o contexto ativo.

- `kubectl config set-context developer --cluster kind-k8s-control-plane --namespace dev --user developer`
  - Cria ou atualiza o contexto `developer`.
  - `--cluster kind-k8s-control-plane`: cluster para este contexto.
  - `--namespace dev`: namespace padrão.
  - `--user developer`: credenciais para usar.

- `kubectl auth can-i --list`
  - Lista todas as permissões do usuário atual no namespace atual.
  - Mostra: Resources, Non-Resource URLs, Resource Names, Verbs.
  - Útil para verificar RBAC: quais recursos/verbs o usuário pode executar.
  - Exemplo: `pods get,list,create` se Role permitir.

- `kubectl apply -f platform.yaml`
  - Aplica o manifesto YAML `platform.yaml` no cluster.
  - Cria um CertificateSigningRequest (CSR) para o usuário `platform`.
  - O CSR contém a solicitação de certificado codificada em base64.
  - Após execução, o CSR fica em estado `Pending` aguardando aprovação.

- `kubectl certificate approve platform`
  - Aprova a CSR chamada `platform`.
  - O controller de certificates no cluster gera o certificado assinado.
  - Após aprovação, a CSR muda para estado `Approved,Issued`.

- `kubectl get csr platform -o jsonpath='{.status.certificate}' | base64 -d > platform.crt`
  - Extrai o certificado assinado do objeto CSR `platform`.
  - `-o jsonpath='{.status.certificate}'`: extrai apenas o campo `.status.certificate` (em base64).
  - `| base64 -d`: decodifica de base64 para formato binário PEM.
  - `> platform.crt`: escreve o certificado decodificado no arquivo `platform.crt`.
  - Resultado: arquivo `platform.crt` contém o certificado X.509 assinado.

- `cp role-developer.yaml role-platform.yaml`
  - Comando shell que copia o arquivo `role-developer.yaml` para `role-platform.yaml`.
  - Permite reutilizar a configuração de Role do usuário developer para o usuário platform.
  - Após cópia, é necessário editar `role-platform.yaml` para mudar metadata.name de `developer` para `platform`.

- `kubectl apply -f clusterrole-platform.yaml`
  - Aplica um ClusterRole para o usuário `platform`.
  - Diferente de Role (namespaced), ClusterRole define permissões em escopo de cluster.
  - Pode conceder acesso a recursos cluster-wide (nodes, clusterroles, etc.) ou all-namespaces.

- `kubectl get clusterrole`
  - Lista todos os ClusterRoles disponíveis no cluster.
  - Mostra: NAME, AGE.
  - Não exige `-n namespace` pois ClusterRoles são cluster-scoped.
  - Exemplo de saída: admin, edit, system:aggregate-to-admin, platform, etc.

- `kubectl apply -f service-account.yaml`
  - Cria ServiceAccount (`pod-leitor`) no namespace atual.
- `kubectl get serviceaccounts`
  - Lista serviceaccounts no namespace.
- `kubectl describe serviceaccounts pod-leitor`
  - Exibe detalhes da SA, incluindo Secrets vinculados.
- `kubectl apply -f service-account-secret.yaml`
  - Cria Secret tipo `kubernetes.io/service-account-token` para a SA.
- `kubectl get secrets service-account-secret`
  - Verifica existência do Secret.
- `kubectl get secrets service-account-secret -o yaml`
  - Exibe:
    - `.data.ca.crt` (CA do cluster, base64)
    - `.data.token` (token JWT, base64)
    - `.data.namespace` (namespace da SA, base64)
    - annotations: `kubernetes.io/service-account.name`, `...uid`
- `kubectl get secret service-account-secret -o jsonpath='{.data.token}' | base64 -d`
  - Extrai token em texto.
- Uso do token:
  - `kubectl --server=<api-server> --certificate-authority=ca.crt --token=<token> get pods`
  - ou:
    - `kubectl config set-credentials pod-leitor --token=<token>`
    - `kubectl config set-context pod-leitor --cluster=<cluster> --namespace=default --user=pod-leitor`
    - `kubectl config use-context pod-leitor`
