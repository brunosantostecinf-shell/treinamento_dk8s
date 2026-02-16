### Configuração Avançada de HPA: Definindo Comportamento de Escalonamento

O HPA é flexível e permite que você defina como ele deve se comportar durante o escalonamento para cima e para baixo. Vamos explorar um exemplo:

```yaml
# Definição de HPA com configurações avançadas de comportamento
apiVersion: autoscaling/v2      # Versão da API que define um HPA
kind: HorizontalPodAutoscaler        # Tipo de recurso que estamos definindo
metadata:
  name: nginx-deployment-hpa         # Nome do nosso HPA
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
      name: cpu                      # Nome da métrica (CPU neste caso)
      target:
        type: Utilization            # Tipo de alvo (utilização)
        averageUtilization: 50       # Valor alvo (50% de utilização)
  behavior:
    scaleUp:
      stabilizationWindowSeconds: 0  # Período de estabilização para escalonamento para cima
      policies:
      - type: Percent                # Tipo de política (percentual)
        value: 100                   # Valor da política (100%)
        periodSeconds: 15            # Período da política (15 segundos)
    scaleDown:
      stabilizationWindowSeconds: 300  # Período de estabilização para escalonamento para baixo
      policies:
      - type: Percent                  # Tipo de política (percentual)
        value: 100                     # Valor da política (100%)
        periodSeconds: 15              # Período da política (15 segundos)
```

Neste exemplo, especificamos um comportamento de escalonamento onde o HPA pode escalar para cima imediatamente, mas vai esperar por 5 minutos (300 segundos) após o último escalonamento para cima antes de considerar um escalonamento para baixo. Isso ajuda a evitar flutuações rápidas na contagem de réplicas, proporcionando um ambiente mais estável para nossos pods.

