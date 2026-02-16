### Autoscaling com base na utilização de Memória

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

