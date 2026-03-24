#### Criar Role e RoleBinding

Crie uma Role chamada "pod-reader" no namespace "dev" que permita listar e obter Pods, e depois crie um RoleBinding que associe essa Role ao usuário "joao".

#### Reposta:

```
kubectl create role pod-reader --verb=get,list --resource=pod -n dev

kubectl create rolebinding pod-reader-binding --role=pod-reader --user=joao --namespace=dev
```

### Criar ClusterRole e ClusterRoleBinding

Crie um ClusterRole chamado "node-viewer" que permita listar, obter e assistir nodes, e um ClusterRoleBinding que o associe ao ServiceAccount "monitoring" no namespace "kube-system".

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources:
  - # Defina o recurso
  verbs:
  - # Defina os verbos
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-viewer-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-viewer
subjects:
- kind: ServiceAccount
  name: # Defina o ServiceAccount
  namespace: # Defina o namespace
```

#### Reposta:

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: node-viewer
rules:
- apiGroups: [""]
  resources:
  - nodes
  verbs:
  - get
  - list
  - watch
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: node-viewer-binding
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: node-viewer
subjects:
- kind: ServiceAccount
  name: monitoring
  namespace: kube-system
```

### Criar ServiceAccount e gerar Token

Crie um ServiceAccount chamado "deploy-bot" no namespace "dev" e gere um token para ele.

#### Reposta:

```
kubectl create serviceaccount deploy-bot -n dev

kubectl create token deploy-bot -n dev
```

#### Verificar permissões com can-i

Use kubectl auth can-i para verificar se o usuário "joao" tem permissão para listar Pods no namespace "dev".

#### Reposta:

```
kubectl auth can-i list pods --namespace=dev --as=joao

kubectl auth can-i delete pods --namespace=dev --as=joao
```