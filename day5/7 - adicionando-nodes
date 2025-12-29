Adicionando os demais nodes ao cluster
Agora que já temos o nosso cluster inicializado e já estamos entendendo muito bem o que é o arquivo admin.conf, chegou o momento de adicionar os demais nodes ao nosso cluster.

Para isso, vamos novamente utilizar o comando kubeadm, porém ao invés de executar o comando no node do control plane, nesse momento precisamos rodar o comando diretamente no node que queremos adicionar ao cluster.

Quando inicializamos o nosso cluster, o kubeadm nos mostrou o comando que precisamos executar no novos nodes, para que eles possam ser adicinados ao cluster como workers.

sudo kubeadm join 172.31.57.89:6443 --token if9hn9.xhxo6s89byj9rsmd \
	--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477 
 

O comando kubeadm join é usado para adicionar um novo node ao cluster Kubernetes existente. Ele é executado nos worker nodes para que eles possam se juntar ao cluster e receber instruções do control plane. Vamos analisar as partes do comando fornecido:

kubeadm join: O comando base para adicionar um novo nó ao cluster.

172.31.57.89:6443: Endereço IP e porta do servidor de API do nó mestre (control plane). Neste exemplo, o nó mestre está localizado no IP 172.31.57.89 e a porta é 6443.

--token if9hn9.xhxo6s89byj9rsmd: O token é usado para autenticar o nó trabalhador no nó mestre durante o processo de adesão. Os tokens são gerados pelo nó mestre e têm uma validade limitada (por padrão, 24 horas). Neste exemplo, o token é if9hn9.xhxo6s89byj9rsmd.

--discovery-token-ca-cert-hash sha256:ad583497a4171d1fc7d21e2ca2ea7b32bdc8450a1a4ca4cfa2022748a99fa477: Este é um hash criptográfico do certificado da autoridade de certificação (CA) do control plane. Ele é usado para garantir que o nó worker esteja se comunicando com o nó do control plane correto e autêntico. O valor após sha256: é o hash do certificado CA.

Ao executar este comando no worker, ele iniciará o processo de adesão ao cluster. Se o token for válido e o hash do certificado CA corresponder ao certificado CA do nó do control plane, o nó worker será autenticado e adicionado ao cluster. Após a adesão bem-sucedida, o novo node começará a executar os Pods e a receber instruções do control plane, conforme necessário.

Após executar o join em cada worker node, vá até o node que criamos para ser o control plane, e execute:

kubectl get nodes
 

NAME     STATUS   ROLES           AGE   VERSION
k8s-01   NotReady    control-plane   4m   v1.26.3
k8s-02   NotReady    <none>          3m   v1.26.3
k8s-03   NotReady    <none>          3m   v1.26.3
 

Agora você já consegue ver que os dois novos nodes foram adicionados ao cluster, porém ainda estão com o status Not Ready, pois ainda não instalamos o nosso plugin de rede para que seja possível a comunicação entre os pods. Vamos resolver isso agora. :)

Instalando o Weave Net
Agora que o cluster está inicializado, vamos instalar o Weave Net:

$ kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml
 

Aguarde alguns minutos até que todos os componentes do cluster estejam em funcionamento. Você pode verificar o status dos componentes do cluster com o seguinte comando:

kubectl get pods -n kube-system
 

kubectl get nodes
 

NAME     STATUS   ROLES           AGE   VERSION
k8s-01   Ready    control-plane   7m   v1.26.3
k8s-02   Ready    <none>          6m   v1.26.3
k8s-03   Ready    <none>          6m   v1.26.3
 

O Weave Net é um plugin de rede que permite que os pods se comuniquem entre si. Ele também permite que os pods se comuniquem com o mundo externo, como outros clusters ou a Internet. Quando o Kubernetes é instalado, ele resolve vários problemas por si só, porém quando o assunto é a comunicação entre os pods, ele não resolve. Por isso, precisamos instalar um plugin de rede para resolver esse problema.

O que é o CNI?
CNI é uma especificação e conjunto de bibliotecas para a configuração de interfaces de rede em containers. A CNI permite que diferentes soluções de rede sejam integradas ao Kubernetes, facilitando a comunicação entre os Pods (grupos de containers) e serviços.

Com isso, temos diferentes plugins de redes, que seguem a especificação CNI, e que podem ser utilizados no Kubernetes. O Weave Net é um desses plugins de rede.

Entre os plugins de rede mais utilizados no Kubernetes, temos:

Calico é um dos plugins de rede mais populares e amplamente utilizados no Kubernetes. Ele fornece segurança de rede e permite a implementação de políticas de rede. O Calico utiliza o BGP (Border Gateway Protocol) para rotear tráfego entre os nós do cluster, proporcionando um desempenho eficiente e escalável.

Flannel é um plugin de rede simples e fácil de configurar, projetado para o Kubernetes. Ele cria uma rede overlay que permite que os Pods se comuniquem entre si, mesmo em diferentes nós do cluster. O Flannel atribui um intervalo de IPs a cada nó e utiliza um protocolo simples para rotear o tráfego entre os nós.

Weave é outra solução popular de rede para Kubernetes. Ele fornece uma rede overlay que permite a comunicação entre os Pods em diferentes nós. Além disso, o Weave suporta criptografia de rede e gerenciamento de políticas de rede. Ele também pode ser integrado com outras soluções, como o Calico, para fornecer recursos adicionais de segurança e políticas de rede.

Cilium é um plugin de rede focado em segurança e desempenho. Ele utiliza o BPF (Berkeley Packet Filter) para fornecer políticas de rede e segurança de alto desempenho. O Cilium também oferece recursos avançados, como balanceamento de carga, monitoramento e solução de problemas de rede.

Kube-router é uma solução de rede leve para Kubernetes. Ele utiliza o BGP e IPVS (IP Virtual Server) para rotear o tráfego entre os nós do cluster, proporcionando um desempenho eficiente e escalável. Kube-router também suporta políticas de rede e permite a implementação de firewalls entre os Pods.

Esses são apenas alguns dos plugins de rede mais populares e amplamente utilizados no Kubernetes. Você pode encontrar uma lista completa de plugins de rede no site do Kubernetes.

Agora, qual você deverá escolher? A resposta é simples: o que melhor se adequar às suas necessidades. Cada plugin de rede tem suas vantagens e desvantagens, e você deve escolher aquele que melhor se adequar ao seu ambiente.

Minha dica, procure não ficar inventando muita moda, tenta utilizar os que são já validados e bem aceitos pela comunidade, como o Weave Net, Calico, Flannel, etc.

O meu preferido é o Weave Net pela simplicidade de instalação e os recursos oferecidos.

Um cara que eu tenho gostado bastante é o Cilium, ele é bem completo e tem uma comunidade bem ativa, além de utilizar o BPF, que é um assunto super quente no mundo Kubernetes!

 

Pronto, já temos o nosso cluster inicializado e o Weave Net instalado. Agora, vamos criar um Deployment para testar a comunicação entre os Pods.

kubectl create deployment nginx --image=nginx --replicas 3
 

kubectl get pods -o wide
 

NAME                     READY   STATUS    RESTARTS   AGE   IP          NODE     NOMINATED NODE   READINESS GATES
nginx-748c667d99-8brrj   1/1     Running   0          12s   10.32.0.4   k8s-02   <none>           <none>
nginx-748c667d99-8knx2   1/1     Running   0          12s   10.40.0.2   k8s-03   <none>           <none>
nginx-748c667d99-l6w7r   1/1     Running   0          12s   10.40.0.1   k8s-03   <none>           <none>
 

Pronto, nosso cluster está funcionando e os Pods estão em execução em diferentes nós.

Agora você já pode se divertir e utilizar o seu mais novo cluster Kubernetes!

