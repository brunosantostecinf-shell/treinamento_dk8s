### Criando a nossa primeira Policy

Kyverno permite que você defina, gerencie e aplique políticas de forma declarativa para garantir que os clusters e suas cargas de trabalho estejam em conformidade com as regras e normas definidas.

As políticas, ou as policies em inglês, do Kyverno podem ser aplicadas de duas maneiras principais: a nível de cluster (ClusterPolicy) ou a nível de namespace específico (Policy).

    1 - ClusterPolicy: Quando você define uma política como ClusterPolicy, ela é aplicada a todos os namespaces no cluster. Ou seja, as regras definidas em uma ClusterPolicy são automaticamente aplicadas a todos os recursos correspondentes em todos os namespaces, a menos que especificamente excluídos.

    2 - Policy: Se você deseja aplicar políticas a um namespace específico, você usaria o tipo Policy. As políticas definidas como Policy são aplicadas apenas dentro do namespace onde são criadas.

Se você não especificar nenhum namespace na política ou usar ClusterPolicy, o Kyverno assumirá que a política deve ser aplicada globalmente, ou seja, em todos os namespaces.

Exemplo de Políticas do Kyverno:

    1- Política de Limites de Recursos: Garantir que todos os containers em um Pod tenham limites de CPU e memória definidos. Isso pode ser importante para evitar o uso excessivo de recursos em um cluster compartilhado.

**Arquivo require-resources-limits.yaml:**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: require-cpu-memory-limits
spec:
  validationFailureAction: enforce
  rules:
  - name: validate-limits
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "CPU and memory limits are required"
      pattern:
        spec:
          containers:
          - name: "*"
            resources:
              limits:
                memory: "?*"
                cpu: "?*"
```

Depois do arquivo criado, agora bastar realizar o deploy em nosso cluster Kubernetes.

```yaml
kubectl apply -f require-resources-limits.yaml
```

Agora, tenta realizar o deploy de um simples Nginx sem definir o limite para os recursos.

Arquivo pod.yaml:

```
apiVersion: v1
kind: Pod
metadata:
  name: exemplo-pod
spec:
  containers:
  - name: exemplo-container
    image: nginx


kubectl apply -f pod.yaml
```

