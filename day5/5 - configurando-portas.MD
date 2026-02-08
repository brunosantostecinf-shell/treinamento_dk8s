### Configurando as portas

Antes de iniciar o cluster, lembre-se das portas que precisam estar abertas para que o cluster funcione corretamente. Precisamos ter as portas TCP 6443, 10250-10255, 30000-32767 e 2379-2380 abertas entre os nodes do cluster. No nosso exemplo, onde estaremos usando somente um node control plane, não precisamos nos preocupar com algumas quando temos mais de um node control plane, pois eles precisam se comunicar entre si para manter o estado do cluster, ou ainda as portas 30000-32767, que são usadas para serviços NodePort que precisam ser acessíveis fora do cluster. Elas nós podemos ir abrindo conforme a necessidade, conforme vamos criando os nossos serviços.

Por agora o que precisamos garantir são as portas TCP 6443 somente no control plane e as 10250-10255 abertas em todos nodes do cluster.

Em nosso exemplo vamos utilizar como CNI o Weave Net, que é um CNI que utiliza o protocolo de roteamento de pacotes do Kubernetes para criar uma rede entre os pods. Falarei mais sobre ele mais pra frente, mas como estamos falando das portas que são importantes para o cluster funcionar, precisamos abrir a porta TCP 6783 e as portas UDP 6783 e 6784, para que o Weave Net funcione corretamente.

Então já sabe, não esqueça de abrir as portas TCP 6443, 10250-10255 e 6783 no seu firewall.

