### Criando um HPA
Antes de nos aprofundarmos no HPA, vamos recapitular criando um deployment simples para o nosso confiável servidor Nginx.

```yaml
# Definição de um Deployment para o servidor Nginx
apiVersion: apps/v1  # Versão da API que define um Deployment
kind: Deployment     # Tipo de recurso que estamos definindo
metadata:
  name: nginx-deployment  # Nome do nosso Deployment
spec:
  replicas: 3             # Número inicial de réplicas
  selector:
    matchLabels:
      app: nginx         # Label que identifica os pods deste Deployment
  template:
    metadata:
      labels:
        app: nginx       # Label aplicada aos pods
    spec:
      containers:
      - name: nginx      # Nome do contêiner
        image: nginx:latest  # Imagem do contêiner
        ports:
        - containerPort: 80  # Porta exposta pelo contêiner
        resources:
          limits:
            cpu: 500m        # Limite de CPU
            memory: 256Mi    # Limite de memória
          requests:
            cpu: 250m        # Requisição de CPU
            memory: 128Mi    # Requisição de memória
```

Agora, com nosso deployment pronto, vamos dar o próximo passo na criação do nosso HPA.

```yaml
# Definição do HPA para o nginx-deployment
apiVersion: autoscaling/v2  # Versão da API que define um HPA
kind: HorizontalPodAutoscaler  # Tipo de recurso que estamos definindo
metadata:
  name: nginx-deployment-hpa  # Nome do nosso HPA
spec:
  scaleTargetRef:
    apiVersion: apps/v1        # A versão da API do recurso alvo
    kind: Deployment           # O tipo de recurso alvo
    name: nginx-deployment     # O nome do recurso alvo
  minReplicas: 3               # Número mínimo de réplicas
  maxReplicas: 10              # Número máximo de réplicas
  metrics:
  - type: Resource             # Tipo de métrica (recurso do sistema)
    resource:
      name: cpu                # Nome da métrica (CPU neste caso)
      target:
        type: Utilization      # Tipo de alvo (utilização)
        averageUtilization: 50 # Valor alvo (50% de utilização)
```

Neste exemplo, criamos um HPA que monitora a utilização da CPU do nosso nginx-deployment. O HPA se esforçará para manter a utilização da CPU em torno de 50%, ajustando o número de réplicas entre 3 e 10 conforme necessário.

Para aplicar esta configuração ao seu cluster Kubernetes, salve o conteúdo acima em um arquivo chamado

nginx-deployment-hpa.yaml e execute o seguinte comando:

```
kubectl apply -f nginx-deployment-hpa.yaml
```

Agora, você tem um HPA monitorando e ajustando a escala do seu nginx-deployment baseado na utilização da CPU. Fantástico, não é?

### Exemplos Práticos com HPA

Agora que você já entende o básico sobre o HPA, é hora de rolar as mangas e entrar na prática. Vamos explorar como o HPA responde a diferentes métricas e cenários.

#### Autoscaling com base na utilização de CPU

Vamos começar com um exemplo clássico de escalonamento baseado na utilização da CPU, que já discutimos anteriormente. Para tornar a aprendizagem mais interativa, vamos simular um aumento de tráfego e observar como o HPA responde a essa mudança.

```
kubectl run -i --tty load-generator --image=busybox /bin/sh

while true; do wget -q -O- http://nginx-deployment.default.svc.cluster.local; done
```

Este script simples cria uma carga constante no nosso deployment, fazendo requisições contínuas ao servidor Nginx. Você poderá observar como o HPA ajusta o número de réplicas para manter a utilização da CPU em torno do limite definido.

#### Autoscaling com base na utilização de Memória
O HPA não é apenas um mestre em lidar com a CPU, ele também tem um olho afiado para a memória. Vamos explorar como configurar o HPA para escalar baseado na utilização de memória.

```yaml
# Definição do HPA para escalonamento baseado em memória
apiVersion: autoscaling/v2  # Versão da API que define um HPA
kind: HorizontalPodAutoscaler    # Tipo de recurso que estamos definindo
metadata:
  name: nginx-deployment-hpa-memory  # Nome do nosso HPA
spec:
  scaleTargetRef:
    apiVersion: apps/v1              # A versão da API do recurso alvo
    kind: Deployment                 # O tipo de recurso alvo
    name: nginx-deployment           # O nome do recurso alvo
  minReplicas: 3                     # Número mínimo de réplicas
  maxReplicas: 10                    # Número máximo de réplicas
  metrics:
  - type: Resource                   # Tipo de métrica (recurso do sistema)
    resource:
      name: memory                   # Nome da métrica (memória neste caso)
      target:
        type: Utilization            # Tipo de alvo (utilização)
        averageUtilization: 70       # Valor alvo (70% de utilização)
```

Neste exemplo, o HPA vai ajustar o número de réplicas para manter a utilização de memória em cerca de 70%. Assim, nosso deployment pode respirar livremente mesmo quando a demanda aumenta.

