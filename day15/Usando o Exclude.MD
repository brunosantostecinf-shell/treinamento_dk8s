### Exemplo de Política: Usando o Exclude

A política require-resources-limits é uma abordagem proativa para gerenciar a utilização de recursos em um cluster Kubernetes. Ela garante que todos os Pods tenham limites de recursos definidos, como CPU e memória, mas com uma exceção específica para um namespace.

**Detalhes da Política**

Essa política impõe que cada Pod no cluster tenha limites explícitos de CPU e memória configurados. Isso é crucial para evitar o consumo excessivo de recursos, que pode afetar outros Pods e a estabilidade geral do cluster. No entanto, esta política exclui especificamente o namespace giropops desta regra.

Arquivo de Política
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-resources-limits
spec:
  validationFailureAction: Enforce
  rules:
  - name: validate-limits
    match:
      resources:
        kinds:
        - Pod
    exclude:
      resources:
        namespaces:
        - giropops
    validate:
      message: "Precisa definir o limites de recursos"
      pattern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                cpu: "?*"
                memory: "?*"

**Implementação e Efeitos**

Ao aplicar esta política, todos os Pods novos ou atualizados precisam ter limites de recursos claramente definidos, exceto aqueles no namespace giropops. Isso assegura uma melhor gestão de recursos e evita situações onde alguns Pods possam monopolizar recursos em detrimento de outros.

