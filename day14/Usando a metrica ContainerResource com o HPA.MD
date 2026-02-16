### ContainerResource

O tipo de métrica ContainerResource no Kubernetes permite que você especifique métricas de recursos específicas do container para escalar. Diferente das métricas de recurso comuns que são aplicadas a todos os contêineres em um Pod, as métricas ContainerResource permitem especificar métricas para um contêiner específico dentro de um Pod. Isso pode ser útil em cenários onde você tem múltiplos contêineres em um Pod, mas quer escalar com base na utilização de recursos de um contêiner específico.

Aqui está um exemplo de como você pode configurar um Horizontal Pod Autoscaler (HPA) usando uma métrica ContainerResource para escalar um Deployment com base na utilização de CPU de um contêiner específico:

```yaml
apiVersion: autoscaling/v2beta2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx
  minReplicas: 3
  maxReplicas: 10
  metrics:
  - type: ContainerResource
    containerResource:
      name: cpu
      container: nginx-NOME-COMPLETO-DO-CONTAINER
      target:
        type: Utilization
        averageUtilization: 50
```

No exemplo acima:

- O tipo de métrica é definido como ContainerResource.
- Dentro do bloco containerResource, especificamos o nome da métrica (cpu), o nome do contêiner (my-container) e o alvo de utilização (averageUtilization: 50).

Isso significa que o HPA vai ajustar o número de réplicas do Deployment nginx para manter a utilização média de CPU do contêiner nginx-NOME-COMPLETO-DO-CONTAINER em torno de 50%.

Este tipo de configuração permite um controle mais granular sobre o comportamento de autoscaling, especialmente em ambientes onde os Pods contêm múltiplos contêineres com diferentes perfis de utilização de recursos.

### Detalhes do Algoritmo de Escalonamento

Cálculo do Número de Réplicas
O núcleo do Horizontal Pod Autoscaler (HPA) é o seu algoritmo de escalonamento, que determina o número ideal de réplicas com base nas métricas fornecidas. A fórmula básica utilizada pelo HPA para calcular o número desejado de réplicas é:

$$
desiredReplicas = \left\lceil currentReplicas \times \left( \frac{currentMetricValue}{desiredMetricValue} \right) \right\rceil
$$

Exemplos com Valores Específicos:

1 - Exemplo de Escala para Cima:
- Réplicas atuais: 2
- Valor atual da métrica (CPU): 80%
- Valor desejado da métrica (CPU): 50%
- Cálculo: ⌈2×(80%/50%)⌉=⌈3.2⌉=4 réplicas

2 - Exemplo de Escala para Baixo:

- Réplicas atuais: 5
- Valor atual da métrica (CPU): 30%
- Valor desejado da métrica (CPU): 50%
- Cálculo: ⌈5×(30%/50%)⌉=⌈3⌉=3 réplicas

Considerações Sobre Métricas e Estado dos Pods:

- Métricas de Recurso por Pod e Personalizadas: O HPA pode ser configurado para usar métricas padrão (como CPU e memória) ou métricas personalizadas definidas pelo usuário, permitindo maior flexibilidade.

- Tratamento de Pods sem Métricas ou Não Prontos: Se um Pod não tiver métricas disponíveis ou não estiver pronto, ele pode ser excluído do cálculo de média, evitando decisões de escalonamento baseadas em dados incompletos.