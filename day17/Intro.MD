### O que iremos ver hoje?

Hoje vamos dedicar o nosso tempo para entender o mundo das Network Policies no Kubernetes. Essa é uma ferramenta essencial para a segurança e o gerenciamento eficaz da comunicação entre os Pods em um cluster Kubernetes. Vamos aprender como as Network Policies funcionam, suas aplicações práticas e como você pode implementá-las para proteger suas aplicações no Kubernetes.
Com certeza será um dia com muito conteúdo e aprendizado. Bora lá?

### O que são Network Policies?

No Kubernetes, uma Network Policy é um conjunto de regras que definem como os Pods podem se comunicar entre si e com outros endpoints de rede. Por padrão, os Pods em um cluster Kubernetes podem se comunicar livremente entre si, o que pode não ser ideal para todos os cenários. As Network Policies permitem que você restrinja esse acesso, garantindo que apenas o tráfego permitido possa fluir entre os Pods ou para/endereços IP externos.

#### Para que Servem as Network Policies?

Network Policies são usadas para:

* Isolar Pods de tráfego não autorizado.
* Controlar o acesso à serviços específicos.
* Implementar padrões de segurança e conformidade.

#### Conceitos Fundamentais: Ingress e Egress

* Ingress: Regras de ingresso controlam o tráfego de entrada para um Pod.
* Egress: Regras de saída controlam o tráfego de saída de um Pod.

Ter o entendimento desses conceitos é fundamental para entender como as Network Policies funcionam, pois você precisará especificar se uma regra se aplica ao tráfego de entrada ou de saída.

#### Como Funcionam as Network Policies?

Network Policies utilizam SELECTORS para identificar grupos de Pods e definir regras de tráfego para eles. A política pode especificar:

* Ingress (entrada): quais Pods ou endereços IP podem se conectar a Pods selecionados.
* Egress (saída): para quais Pods ou endereços IP os Pods selecionados podem se conectar.

#### Ainda não é padrão

Infelizmente, as Network Policies ainda não são um recurso padrão em todos os clusters Kubernetes. Recentemente a AWS anunciou o suporte a Network Policies no EKS, mas ainda não é um recurso padrão, para que você possa utilizar as Network Policies no EKS, você precisa instalar o CNI da AWS e depois habilitar o Network Policy nas configurações avançadas do CNI.

Para verificar se o seu cluster suporta Network Policies, você pode executar o seguinte comando:

```bash
kubectl api-versions | grep networking
```

Se você receber a mensagem networking.k8s.io/v1, significa que o seu cluster suporta Network Policies. Se você receber a mensagem networking.k8s.io/v1beta1, significa que o seu cluster não suporta Network Policies.

Se o seu cluster não suportar Network Policies, você pode utilizar o Calico para implementar as Network Policies no seu cluster. Para isso, você precisa instalar o Calico no seu cluster. Você pode encontrar mais informações sobre o Calico aqui.

Outros CNI que suportam Network Policies são o Weave Net e o Cilium, por exemplo.

