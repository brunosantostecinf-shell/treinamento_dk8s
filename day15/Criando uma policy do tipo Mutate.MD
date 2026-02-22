### Exemplo de Política: Adicionar Label ao Namespace

A política add-label-namespace é projetada para automatizar a adição de um label específico a todos os Namespaces em um cluster Kubernetes. Esta abordagem é essencial para a organização, monitoramento e controle de acesso em ambientes complexos.

Detalhes da Política
O label adicionado por esta política é Jeferson: "Lindo_Demais". A aplicação deste label a todos os Namespaces facilita a identificação e a categorização dos mesmos, permitindo uma gestão mais eficiente e uma padronização no uso de labels.

**Arquivo de Política: add-label-namespace.yaml**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: add-label-namespace
spec:
  rules:
  - name: add-label-ns
    match:
      resources:
        kinds:
        - Namespace
    mutate:
      patchStrategicMerge:
        metadata:
          labels:
            Jeferson: "Lindo_Demais"
```

Utilização da Política

Esta política garante que cada Namespace no cluster seja automaticamente etiquetado com Jeferson: "Lindo_Demais". Isso é particularmente útil para garantir a conformidade e a uniformidade na atribuição de labels, facilitando operações como filtragem e busca de Namespaces com base em critérios específicos.