Hoje focamos em descomplicar as Network Policies no Kubernetes. Network Policies são conjuntos de regras que definem como os Pods se comunicam entre si e com outros endpoints de rede. Essenciais para a segurança, elas controlam o acesso a serviços e implementam padrões de conformidade, sendo fundamentais para isolar Pods de tráfego não autorizado. Abordamos conceitos de Ingress (tráfego de entrada) e Egress (tráfego de saída), fundamentais para o entendimento das Network Policies.

Detalhamos o processo de criação de um Cluster EKS (Kubernetes gerenciado pela AWS), destacando as facilidades que ele oferece em termos de gerenciamento e configuração. Analisamos diferentes tipos de clusters EKS: Managed Node Groups, Self-Managed Node Groups e Fargate, cada um com seus prós e contras. O foco foi no Managed Node Groups para simplificar a administração dos Nodes Workers.

Instruímos sobre a instalação do EKSCTL e AWS CLI, ferramentas essenciais para a criação e gerenciamento de clusters EKS. Através de exemplos práticos, demonstramos como criar um cluster EKS, instalar o AWS VPC CNI Plugin e habilitar Network Policies nas configurações avançadas do CNI.

Introduzimos o Nginx Ingress Controller, essencial para o funcionamento da nossa aplicação de exemplo, o Giropops-Senhas, que utiliza Flask e Redis. Explicamos como criar e aplicar regras de Network Policy para controlar o acesso a essa aplicação dentro de um namespace Kubernetes.

Finalmente, discutimos a importância de regras de Egress para controlar o tráfego de saída de Pods e garantir a segurança e eficiência da comunicação em um cluster Kubernetes. Demonstramos como as Network Policies podem ser usadas para criar um ambiente de rede seguro e controlado, essencial para qualquer aplicação executada em Kubernetes.

