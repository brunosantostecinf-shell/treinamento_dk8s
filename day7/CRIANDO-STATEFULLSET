Criando um StatefulSet
Para a criação de um StatefulSet precisamos de um arquivo de configuração que descreva o StatefulSet que queremos criar. Não é possível criar um StatefulSet sem um manifesto yaml, diferente do que acontece com o Pod e o Deployment.

Para o nosso exemplo, vamos utilizar o Nginx como aplicação que será gerenciada pelo StatefulSet, e vamos fazer com que cada Pod tenha um volume persistente associado a ele, e com isso, vamos ter uma página web diferente para cada Pod.

Para isso, vamos criar o arquivo nginx-statefulset.yaml com o seguinte conteúdo:

apiVersion: apps/v1
kind: StatefulSet # Tipo do recurso que estamos criando, no caso, um StatefulSet
metadata:
  name: nginx
spec:
  serviceName: "nginx"
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates: # Como estamos utilizando StatefulSet, precisamos criar um template de volume para cada Pod, entã ao invés de criarmos um volume diretamente, criamos um template que será utilizado para criar um volume para cada Pod
  - metadata:
      name: www # Nome do volume, assim teremos o volume www-0, www-1 e www-2
    spec:
      accessModes: [ "ReadWriteOnce" ]
      resources:
        requests:
          storage: 1Gi
 

Agora, vamos criar o StatefulSet com o comando:

kubectl apply -f nginx-statefulset.yaml
 

Para verificar se o StatefulSet foi criado, podemos utilizar o comando:

kubectl get statefulset
 

Caso queira ver com mais detalhes, podemos utilizar o comando:

kubectl describe statefulset nginx
 

Para verificar se os Pods foram criados, podemos utilizar o comando:

kubectl get pods

