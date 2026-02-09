Storage Class

Uma StorageClass no Kubernetes é um objeto que descreve e define diferentes classes de armazenamento disponíveis no cluster. Essas classes de armazenamento podem ser usadas para provisionar dinamicamente PersistentVolumes (PVs) de acordo com os requisitos dos PersistentVolumeClaims (PVCs) criados pelos usuários.

A StorageClass é útil para gerenciar e organizar diferentes tipos de armazenamento, como armazenamento em disco rápido e caro ou armazenamento em disco mais lento e barato. Além disso, a StorageClass pode ser usada para definir diferentes políticas de retenção, provisionamento e outras características de armazenamento específicas.

Os administradores do cluster podem criar e gerenciar várias StorageClasses para permitir que os usuários finais escolham a classe de armazenamento adequada para suas necessidades.

Cada StorageClass é definida com um provisionador, que é responsável por criar PersistentVolumes dinamicamente conforme necessário. Os provisionadores podem ser internos (fornecidos pelo próprio Kubernetes) ou externos (fornecidos por provedores de armazenamento específicos).

Inclusive os provisionadores podem ser diferentes para cada provedor de nuvem ou onde o Kubernetes está sendo executado. Vou listar alguns provisionadores que são usados e seus respectivos provedores:

- kubernetes.io/aws-ebs: AWS Elastic Block Store (EBS)
- kubernetes.io/azure-disk: Azure Disk
- kubernetes.io/gce-pd: Google Compute Engine (GCE) Persistent Disk
- kubernetes.io/cinder: OpenStack Cinder
- kubernetes.io/vsphere-volume: vSphere
- kubernetes.io/no-provisioner: Volumes locais
- kubernetes.io/host-path: Volumes locais

E se você estiver usando o Kubernetes em um ambiente local, como o Minikube, o provisionador padrão é o kubernetes.io/host-path, que cria volumes PersistentVolume no diretório do host. Já no Kind, o provisionador padrão é o rancher.io/local-path, que cria volumes PersistentVolume no diretório do host.

Para ver a lista completa de provisionadores, consulte a documentação do Kubernetes no link 

https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner.


Para você ver os Storage Classes disponíveis no seu cluster, basta executar o seguinte comando:

```
kubectl get storageclass
 

NAME                 PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
standard (default)   rancher.io/local-path   Delete          WaitForFirstConsumer   false                  21m
``` 

Como você pode ver, no Kind, o provisionador padrão é o rancher.io/local-path, que cria volumes PersistentVolume no diretório do host.

Já no EKS, o provisionador padrão é o kubernetes.io/aws-ebs, que cria volumes PersistentVolume no EBS da AWS.

```
NAME            PROVISIONER             RECLAIMPOLICY   VOLUMEBINDINGMODE      ALLOWVOLUMEEXPANSION   AGE
gp2 (default)   kubernetes.io/aws-ebs   Delete          WaitForFirstConsumer   false                  6h5m
``` 

Vamos ver os detalhes do nosso Storage Class padrão:

```
kubectl describe storageclass standard
 

Name:            standard
IsDefaultClass:  Yes
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{"storageclass.kubernetes.io/is-default-class":"true"},"name":"standard"},"provisioner":"rancher.io/local-path","reclaimPolicy":"Delete","volumeBindingMode":"WaitForFirstConsumer"}
,storageclass.kubernetes.io/is-default-class=true
Provisioner:           rancher.io/local-path
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Delete
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
``` 

Uma coisa que podemos ver é que o nosso Storage Class está com a opção IsDefaultClass como Yes, o que significa que ele é o Storage Class padrão do nosso cluster, com isso todos os Persistent Volume Claims que não tiverem um Storage Class definido, irão utilizar esse Storage Class como padrão.


Vamos criar um novo Storage Class para o nosso cluster Kubernetes no kind, com o nome "local-storage", e vamos definir o provisionador como "kubernetes.io/host-path", que cria volumes PersistentVolume no diretório do host.

```
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: giropops
provisioner: kubernetes.io/no-provisioner
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
 

kubectl apply -f storageclass.yaml
 

storageclass.storage.k8s.io/giropops created
``` 

Pronto! Agora nós temos um novo Storage Class criado no nosso cluster Kubernetes no kind, com o nome "giropops", e com o provisionador "kubernetes.io/no-provisioner", que cria volumes PersistentVolume no diretório do host.

Para saber mais detalhes sobre o Storage Class que criamos, execute o seguinte comando:

```
kubectl describe storageclass giropops 

Name:            giropops
IsDefaultClass:  No
Annotations:     kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"storage.k8s.io/v1","kind":"StorageClass","metadata":{"annotations":{},"name":"giropops"},"provisioner":"kubernetes.io/no-provisioner","reclaimPolicy":"Retain","volumeBindingMode":"WaitForFirstConsumer"}

Provisioner:           kubernetes.io/no-provisioner
Parameters:            <none>
AllowVolumeExpansion:  <unset>
MountOptions:          <none>
ReclaimPolicy:         Retain
VolumeBindingMode:     WaitForFirstConsumer
Events:                <none>
``` 

Lembrando que criamos esse Storage Class com o provisionador "kubernetes.io/no-provisioner", mas você pode criar um Storage Class com qualquer provisionador que você quiser, como o "kubernetes.io/aws-ebs", que cria volumes PersistentVolume no EBS da AWS.
