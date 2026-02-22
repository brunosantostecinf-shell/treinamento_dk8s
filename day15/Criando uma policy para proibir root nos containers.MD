### Exemplo de Política: Proibir Usuário Root

A política disallow-root-user é uma regra de segurança crítica no gerenciamento de clusters Kubernetes. Ela proíbe a execução de containers como usuário root dentro de Pods. Este controle ajuda a prevenir possíveis vulnerabilidades de segurança e a reforçar as melhores práticas no ambiente de contêineres.

**Detalhes da Política**

O principal objetivo desta política é garantir que nenhum Pod no cluster execute containers como o usuário root. A execução de containers como root pode expor o sistema a riscos de segurança, incluindo acessos não autorizados e potenciais danos ao sistema host.

**Arquivo de Política: disallow-root-user.yaml**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: disallow-root-user
spec:
  validationFailureAction: Enforce
  rules:
  - name: check-runAsNonRoot
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Running as root is not allowed. Set runAsNonRoot to true."
      pattern:
        spec:
          containers:
          - securityContext:
              runAsNonRoot: true
```

**Implementação e Efeito**

Ao aplicar esta política, todos os Pods que tentarem executar containers como usuário root serão impedidos, com a exibição de uma mensagem de erro indicando que a execução como root não é permitida. Isso assegura uma camada adicional de segurança no ambiente Kubernetes, evitando práticas que possam comprometer a integridade e a segurança do cluster.