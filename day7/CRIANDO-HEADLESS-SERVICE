O nosso StatefulSet está criado, mas ainda temos que criar o Headless Service para que possamos acessar os Pods individualmente, e para isso, vamos criar o arquivo nginx-headless-service.yaml com o seguinte conteúdo:

apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None # Como estamos criando um Headless Service, não queremos que ele tenha um IP, então definimos o clusterIP como None
  selector:
    app: nginx
 

Pronto, arquivo criado, hora de criar o Headless Service com o comando:

kubectl apply -f nginx-headless-service.yaml
 

Para verificar se o Headless Service foi criado, podemos utilizar o comando:

kubectl get service
 

Caso queira ver com mais detalhes, podemos utilizar o comando:

kubectl describe service nginx
 

Agora que já temos o StatefulSet e o Headless Service criados, podemos acessar cada Pod individualmente, para isso, vamos utilizar o comando:

kubectl run -it --rm debug --image=busybox --restart=Never -- sh
 

Agora vamos utilizar o comando nslookup para verificar o endereço IP de cada Pod, para isso, vamos utilizar o comando:

nslookup nginx-0.nginx.default.svc.cluster.local
 

Agora vamos acessar o Pod utilizando o endereço IP, para isso, vamos utilizar o comando:

wget -O- http://<endereço-ip-do-pod>
 

Precisamos mudar a página web de cada Pod para que possamos verificar se estamos acessando o Pod correto, para isso, vamos utilizar o comando:

echo "Pod 0" > /usr/share/nginx/html/index.html
 

Agora vamos acessar o Pod novamente, para isso, vamos utilizar o comando:

wget -O- http://<endereço-ip-do-pod>
 

A saída do comando deve ser:

Connecting to <endereço-ip-do-pod>:80 (<endereço-ip-do-pod>:80)
Pod 0
 

Caso queira, você pode fazer o mesmo para os outros Pods, basta mudar o número do Pod no comando nslookup e no comando echo.


