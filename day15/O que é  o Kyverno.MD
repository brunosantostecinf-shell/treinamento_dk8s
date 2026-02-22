### Kyverno - Como fazer o seu cluster Kubernetes mais seguro

Kyverno é uma ferramenta de gerenciamento de políticas para Kubernetes, focada na automação de várias tarefas relacionadas à segurança e configuração dos clusters de Kubernetes. Ele permite que você defina, gerencie e aplique políticas de forma declarativa para garantir que os clusters e suas cargas de trabalho estejam em conformidade com as regras e normas definidas.

Principais Funções do Kyverno:

    1 -Validação de Recursos: Verifica se os recursos do Kubernetes estão em conformidade com as políticas definidas. Por exemplo, pode garantir que todos os Pods tenham limites de CPU e memória definidos.

    2- Mutação de Recursos: Modifica automaticamente os recursos do Kubernetes para atender às políticas definidas. Por exemplo, pode adicionar automaticamente labels específicos a todos os novos Pods.

    3- Geração de Recursos: Cria recursos adicionais do Kubernetes com base nas políticas definidas. Por exemplo, pode gerar NetworkPolicies para cada novo Namespace criado.

