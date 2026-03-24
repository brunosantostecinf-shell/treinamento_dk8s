### Criar NetworkPolicy de Ingress

Crie uma NetworkPolicy que permita tráfego de entrada (Ingress) para os Pods com label "app=api" somente a partir de Pods com label "app=frontend" na porta 8080.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      # Selecione os Pods alvo
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          # Selecione os Pods de origem
    ports:
    - port: # Defina a porta
```

### NetworkPolicy com Egress e regra DNS

Crie uma NetworkPolicy que bloqueie todo egress dos Pods com label "app=backend", exceto DNS (porta 53 UDP) e conexões para Pods com label "app=database" na porta 5432.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  # Permita DNS (porta 53 UDP) e conexão com database (porta 5432)
  ```

### NetworkPolicy Deny All Ingress

Crie uma NetworkPolicy que bloqueie TODO o tráfego de entrada para todos os Pods do namespace "production".

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  # Configure para negar todo Ingress
```

### Isolamento de namespace com NetworkPolicy

Crie uma NetworkPolicy que permita tráfego Ingress para Pods com label "app=api" somente de Pods que estejam no namespace com label "team=frontend".

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend-ns
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          # Selecione o namespace de origem
```

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
    ports:
    - protocol: TCP
      port: 8080

 apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: backend-egress
spec:
  podSelector:
    matchLabels:
      app: backend
  policyTypes:
  - Egress
  egress:
  - ports:
    - protocol: UDP
      port: 53
  - to:
    - podSelector:
        matchLabels:
          app: database
    ports:
    - protocol: TCP
      port: 5432     

 apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-ingress
  namespace: production
spec:
  podSelector: {}
  policyTypes:
  - Ingress     

apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-from-frontend-ns
spec:
  podSelector:
    matchLabels:
      app: api
  policyTypes:
  - Ingress
  ingress:
  - from:
    - namespaceSelector:
        matchLabels:
          team: frontend