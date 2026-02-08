Girus Lab - Case-01

Copie e cole o conteúdo abaixo em um arquivo, por exemplo, case-01.yaml.

Depois é só importar lá no Girus com:

```
girus create lab -f case-01.yaml
```

Depois é só acessar o Girus e procurar pelo lab Caso 1: O Deploy de Sexta-Feira e praticar!

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: kubernetes-deploy-sexta-feira-lab
  namespace: girus
  labels:
    app: girus-lab-template
data:
  lab.yaml: |
    name: kubernetes-deploy-sexta-feira
    title: "Caso 1: O Deploy de Sexta-Feira"
    description: "Sexta-feira, 4 da tarde. Uma alteração 'simples' derrubou a produção. Neste laboratório interativo, você vai vivenciar um cenário real de troubleshooting no Kubernetes, aprendendo a diagnosticar problemas de OOMKilled, Probes falhando e estratégias de rollout. Prepare-se para resolver o mistério do deploy que arruinou o fim de semana!"
    duration: 30m
    image: "linuxtips/girus-kind-single-node:0.1"
    privileged: true
    tasks:
      - name: "O Cenário: 'É Só Uma Linha de Código'"
        description: "Simule o deploy fatídico de sexta-feira e observe o caos se instalar no cluster."
        steps:
          - "**Sexta-feira, 4 da tarde.**"
          - "O time de produto pediu uma alteração pequena, quase cosmética, em um dos microsserviços. 'É só mudar um texto, super rápido', eles disseram."
          - "Você fez a alteração, rodou os testes locais, tudo verde. Parece seguro."
          - "Vamos criar o namespace para nossa simulação:"
          - "`kubectl create namespace producao`"
          - "Agora, vamos simular o cenário inicial: um Deployment **sem boas práticas** que estava funcionando 'na sorte'. Crie o arquivo `deployment-problematico.yaml`:"
          - |
            ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: meu-servico
              namespace: producao
            spec:
              replicas: 3
              selector:
                matchLabels:
                  app: meu-servico
              template:
                metadata:
                  labels:
                    app: meu-servico
                spec:
                  containers:
                  - name: meu-servico-container
                    image: nginx:alpine
                    # PROBLEMA 1: Sem resources (requests/limits)
                    # PROBLEMA 2: Sem probes (liveness/readiness/startup)
                    # PROBLEMA 3: Deployment criado há muito tempo, sem seguir boas práticas
            ```
          - "Aplique o manifesto problemático:"
          - "`kubectl apply -f deployment-problematico.yaml`"
          - "Aguarde os pods ficarem prontos:"
          - "`kubectl get pods -n producao -w`"
          - "Pressione Ctrl+C quando os 3 pods estiverem Running."
          - "Tudo parece bem... por enquanto. Você abre o terminal, confiante, e se prepara para o 'pequeno deploy'."
          - "**Mas então, seu celular vibra. E vibra de novo. É o PagerDuty.**"
          - "O canal de alertas do time começa a piscar em vermelho no Slack. O dashboard do Grafana, antes um mar de tranquilidade verde, agora parece uma árvore de natal desgovernada."
          - "Seu colega manda uma mensagem: *'Cara, o serviço X caiu depois do seu deploy'*."
          - "Aquele café que você ia tomar vai ter que esperar."
        tips:
          - type: "warning"
            title: "Dívida Técnica à Vista"
            content: "Este Deployment foi criado sem as garantias mínimas de estabilidade que o Kubernetes oferece. Sem resources e probes, a aplicação está rodando 'na sorte'."
          - type: "info"
            title: "O que aconteceu?"
            content: "Como uma mudança de uma linha de texto derrubou um serviço inteiro? Vamos descobrir seguindo um roteiro lógico de troubleshooting."
        validation:
          - command: "kubectl get deployment meu-servico -n producao -o jsonpath='{.status.readyReplicas}' 2>/dev/null || echo '0'"
            expectedOutput: "3"
            errorMessage: "O Deployment inicial não está com 3 réplicas prontas. Verifique se aplicou o manifesto corretamente."

      - name: "Passo 1: Qual o Estado da Aplicação?"
        description: "Respira fundo e começa do básico. O primeiro comando é sempre kubectl get pods - nossa visão geral do campo de batalha."
        steps:
          - "**Nessas horas, o pânico não ajuda.**"
          - "A primeira regra de um bom profissional SRE ou Dev é: **respira fundo e começa do básico**. Não adianta chutar. Precisamos de dados."
          - "Antes de investigar, vamos simular o problema. Crie uma nova versão 'quebrada' do deployment que consome muita memória. Crie o arquivo `deployment-quebrado.yaml`:"
          - |
            ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: meu-servico
              namespace: producao
            spec:
              replicas: 3
              selector:
                matchLabels:
                  app: meu-servico
              strategy:
                type: RollingUpdate
                rollingUpdate:
                  maxUnavailable: 1
                  maxSurge: 1
              template:
                metadata:
                  labels:
                    app: meu-servico
                    version: "v2-problematica"
                spec:
                  containers:
                  - name: meu-servico-container
                    image: linuxtips/memory-eater:0.1
                    # Simulando limites muito baixos para forçar OOMKill
                    resources:
                      limits:
                        memory: "32Mi"
                    # Sem probes = Kubernetes operando às cegas
            ```
          - "Aplique o deploy problemático:"
          - "`kubectl apply -f deployment-quebrado.yaml`"
          - "Agora, observe o campo de batalha com o comando mais importante:"
          - "`kubectl get pods -n producao`"
          - "Você provavelmente verá algo assim:"
          - "- Alguns pods da versão antiga ainda no ar (Running)"
          - "- Pods da nova versão em **CrashLoopBackOff** ou **OOMKilled**"
          - "- Talvez algum Pod em **Pending**"
          - "Aguarde alguns segundos e observe novamente:"
          - "`kubectl get pods -n producao -w`"
          - "Pressione Ctrl+C após observar o comportamento."
          - "**Diagnóstico inicial:** O problema está claramente na nova versão. Os pods antigos funcionam, os novos não."
        tips:
          - type: "info"
            title: "kubectl get pods - Seu Primeiro Contato"
            content: "Este comando é sempre o primeiro passo. Ele mostra o estado geral dos seus Pods: Running, Pending, CrashLoopBackOff, Error, etc. É a 'visão do campo de batalha'."
          - type: "tip"
            title: "Identificando Versões pelos Hashes"
            content: "O nome do Pod inclui um hash do ReplicaSet (ex: meu-servico-5f... vs meu-servico-7d...). Pods com hashes diferentes pertencem a versões/ReplicaSets diferentes."
          - type: "warning"
            title: "CrashLoopBackOff"
            content: "Este status indica que o container está reiniciando repetidamente. O Kubernetes aumenta o tempo entre as tentativas (backoff) para não sobrecarregar o sistema."
        validation:
          - command: "[ $(kubectl get pods -n producao 2>/dev/null | grep -E 'CrashLoopBackOff|OOMKilled|Error' | wc -l | tr -d ' ') -ge 1 ] && echo '>= 1'|| echo '0'"
            expectedOutput: ">= 1"
            errorMessage: "Nenhum Pod problemático encontrado. Aplique o deployment-quebrado.yaml e aguarde alguns segundos."

      - name: "Passo 2: Aprofundando nos Logs e Eventos"
        description: "Agora que temos um suspeito, vamos interrogá-lo. O kubectl describe pod é nossa principal ferramenta de investigação."
        steps:
          - "**Hora de interrogar o suspeito!**"
          - "Primeiro, vamos pegar o nome de um dos Pods problemáticos:"
          - "`POD_PROBLEMA=$(kubectl get pods -n producao -l version=v2-problematica -o jsonpath='{.items[0].metadata.name}' 2>/dev/null)`"
          - "Agora, usamos o **kubectl describe** para ver o histórico completo do Pod:"
          - "`kubectl describe pod $POD_PROBLEMA -n producao`"
          - "**O que procurar na saída:**"
          - "1. **Seção 'State' e 'Last State'**: Mostra o estado atual e o anterior do container"
          - "2. **Reason**: Procure por `OOMKilled` - significa que o processo foi morto por excesso de memória"
          - "3. **Exit Code**: Código diferente de 0 indica falha"
          - "4. **Seção 'Events'**: A linha do tempo do crime - mostra o histórico de eventos"
          - "Vamos filtrar especificamente os eventos:"
          - "`kubectl describe pod $POD_PROBLEMA -n producao | grep -A 20 'Events:'`"
          - "Você deve ver eventos como:"
          - "- `Warning Unhealthy` - Readiness probe failed"
          - "- `Warning BackOff` - Back-off restarting failed container"
          - "- `Normal Killing` - Container sendo morto"
          - "Vamos verificar os logs do container (se houver):"
          - "`kubectl logs $POD_PROBLEMA -n producao --previous 2>/dev/null || echo 'Sem logs anteriores disponíveis'`"
          - "**Descoberta:** A aplicação está consumindo mais memória do que o permitido (OOMKilled) e não há probes configuradas para o Kubernetes saber se ela está saudável!"
        tips:
          - type: "info"
            title: "OOMKilled - Out of Memory Killed"
            content: "Quando um container excede o limite de memória definido, o Kernel do Linux o mata imediatamente. É uma proteção do sistema para evitar que um processo consuma toda a memória do Node."
          - type: "warning"
            title: "Exit Code 137"
            content: "O código de saída 137 (128 + 9) indica que o processo recebeu um SIGKILL. Isso geralmente significa OOMKilled ou o container foi forçosamente terminado."
          - type: "tip"
            title: "Flag --previous"
            content: "Use `kubectl logs --previous` para ver os logs da execução anterior do container. Essencial quando o container atual está reiniciando constantemente."
        validation:
          - command: "[ $(kubectl describe pod -n producao -l version=v2-problematica 2>/dev/null | grep -c 'OOMKilled\\|BackOff\\|Unhealthy') -ge 1 ] && echo '>= 1' || echo '0'"
            expectedOutput: ">= 1"
            errorMessage: "Não foi possível encontrar evidências de OOMKilled ou falhas. Verifique se o Pod problemático existe."

      - name: "Passo 3: Como Foi a Transição?"
        description: "Um Deployment não gerencia Pods diretamente - ele usa ReplicaSets. Entender o que aconteceu com eles nos ajuda a entender o rollout."
        steps:
          - "**Entendendo a Hierarquia: Deployment → ReplicaSet → Pods**"
          - "Um conceito fundamental: cada vez que você faz um deploy, um **novo ReplicaSet** é criado."
          - "Vamos ver todos os ReplicaSets do nosso Deployment:"
          - "`kubectl get replicaset -n producao`"
          - "Você deve ver algo como:"
          - "```"
          - "NAME DESIRED CURRENT READY AGE"
          - "meu-servico-5f4d... 2 2 2 10m # Versão antiga"
          - "meu-servico-7d8e... 3 3 0 2m # Versão nova (problemática)"
          - "```"
          - "**Análise:**"
          - "- O ReplicaSet antigo ainda tem pods Ready (sobreviventes)"
          - "- O ReplicaSet novo tem pods DESIRED mas 0 READY"
          - "- O Kubernetes está tentando criar os novos pods, mas eles morrem"
          - "Vamos ver mais detalhes:"
          - "`kubectl describe replicaset -n producao -l version=v2-problematica`"
          - "Na seção Events, você verá o ReplicaSet tentando criar pods repetidamente."
          - "**O ReplicaSet está fazendo seu trabalho**. Ele tenta manter o número de réplicas desejado. O problema é que os Pods não colaboram - eles morrem assim que sobem."
        tips:
          - type: "info"
            title: "A Hierarquia K8s"
            content: "Deployment gerencia ReplicaSets, que gerenciam Pods. O Deployment cuida das atualizações e rollbacks. O ReplicaSet garante o número de réplicas. Os Pods executam os containers."
          - type: "tip"
            title: "Por que múltiplos ReplicaSets?"
            content: "Durante um rolling update, o Kubernetes mantém o ReplicaSet antigo (para rollback) enquanto cria o novo. É por isso que você vê dois ReplicaSets para o mesmo Deployment."
          - type: "warning"
            title: "Rolling Update Travado"
            content: "Se os novos Pods não ficam Ready, o rolling update fica travado. O Kubernetes não vai matar todos os Pods antigos até que os novos estejam funcionando."
        validation:
          - command: "[ $(kubectl get replicaset -n producao --no-headers 2>/dev/null | wc -l | tr -d ' ') -ge 2 ] && echo '>= 2' || echo '0'"
            expectedOutput: ">= 2"
            errorMessage: "Deve haver pelo menos 2 ReplicaSets (versão antiga e nova). Verifique se o deploy foi aplicado."

      - name: "Passo 4: Revisando o Manifesto (A Causa Raiz)"
        description: "Se os Pods estão com problemas, a causa raiz está na especificação - o 'DNA' deles. Vamos examinar o Deployment."
        steps:
          - "**Analisando o DNA do Problema**"
          - "Vamos olhar a configuração completa do Deployment:"
          - "`kubectl describe deployment meu-servico -n producao`"
          - "Agora, vamos extrair especificamente a especificação do container:"
          - "`kubectl get deployment meu-servico -n producao -o yaml | grep -A 30 'containers:'`"
          - "**Os Problemas Encontrados:**"
          - "1. **Falta de Resources adequados**: O limite de memória está muito baixo (32Mi), causando OOMKill"
          - "2. **Ausência de Probes**: Sem liveness, readiness ou startup probes"
          - "3. **Sem requests definidos**: O Scheduler não sabe quanto recurso a aplicação precisa"
          - "Vamos confirmar a ausência de probes:"
          - "`kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].livenessProbe}' && echo ' <- Liveness' || echo 'Sem Liveness Probe'`"
          - "`kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].readinessProbe}' && echo ' <- Readiness' || echo 'Sem Readiness Probe'`"
          - "**A Verdade:** Este serviço foi criado há muito tempo, provavelmente por alguém que não seguiu as melhores práticas. Ele sempre funcionou 'na sorte'. A pequena mudança de código aumentou o consumo de memória, e foi **a gota d'água** que fez o castelo de cartas desmoronar."
        tips:
          - type: "warning"
            title: "Dívida Técnica que Venceu"
            content: "O problema não foi a sua linha de código. Foi uma dívida técnica esperando para ser cobrada. A aplicação estava rodando sem as garantias mínimas de estabilidade."
          - type: "info"
            title: "A Combinação Fatal"
            content: "1) Sem requests/limits = bomba-relógio de memória. 2) Sem probes = Kubernetes operando às cegas. 3) RollingUpdate sem readiness = caos durante deploys."
          - type: "tip"
            title: "Prevenção"
            content: "Use ferramentas como OPA Gatekeeper ou Kyverno para bloquear Deployments sem resources e probes em produção."
        validation:
          - command: "kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].resources.limits.memory}' 2>/dev/null"
            expectedOutput: "32Mi"
            errorMessage: "Verifique se o Deployment problemático está aplicado com o limite de memória baixo."

      - name: "A Solução: O Hotfix Salvador"
        description: "Hora de corrigir o problema da forma certa, aplicando todo o conhecimento de resources, probes e estratégias de rollout."
        steps:
          - "**Hora de Corrigir o Problema da Forma Certa!**"
          - "Você cria um novo branch: `hotfix/add-resources-and-probes`"
          - "Crie o arquivo `deployment-corrigido.yaml` com um manifesto robusto:"
          - |
            ```yaml
            apiVersion: apps/v1
            kind: Deployment
            metadata:
              name: meu-servico
              namespace: producao
            spec:
              replicas: 3
              strategy:
                type: RollingUpdate
                rollingUpdate:
                  maxUnavailable: 1 # No máximo 1 pod indisponível
                  maxSurge: 1 # No máximo 1 pod extra durante update
              selector:
                matchLabels:
                  app: meu-servico
              template:
                metadata:
                  labels:
                    app: meu-servico
                    version: "v3-corrigida"
                spec:
                  containers:
                  - name: meu-servico-container
                    image: nginx:alpine
                    ports:
                    - containerPort: 80
                    # CORREÇÃO 1: Definindo uma "cota" de recursos justa
                    resources:
                      requests:
                        cpu: "100m"
                        memory: "128Mi"
                      limits:
                        cpu: "200m"
                        memory: "256Mi"
                    # CORREÇÃO 2: Dando tempo para a aplicação iniciar
                    startupProbe:
                      httpGet:
                        path: /
                        port: 80
                      failureThreshold: 30
                      periodSeconds: 10
                    # CORREÇÃO 3: Checando se a aplicação não travou
                    livenessProbe:
                      httpGet:
                        path: /
                        port: 80
                      initialDelaySeconds: 10
                      periodSeconds: 10
                    # CORREÇÃO 4: Verificando se está pronta para o trabalho
                    readinessProbe:
                      httpGet:
                        path: /
                        port: 80
                      initialDelaySeconds: 5
                      periodSeconds: 5
            ```
          - "Aplique o hotfix:"
          - "`kubectl apply -f deployment-corrigido.yaml`"
          - "Desta vez, observe o rollout de forma controlada:"
          - "`kubectl rollout status deployment/meu-servico -n producao`"
          - "Verifique os pods subindo um a um, de forma ordenada:"
          - "`kubectl get pods -n producao -w`"
          - "Pressione Ctrl+C quando todos os pods estiverem Running e Ready (1/1)."
          - "**O dashboard volta a ficar verde. Você respira aliviado.**"
        tips:
          - type: "success"
            title: "🎉 Problema Resolvido!"
            content: "Os pods agora têm resources adequados, probes configuradas e a estratégia de rollout funciona corretamente porque o Kubernetes sabe quando um pod está pronto."
          - type: "info"
            title: "O Papel de Cada Probe"
            content: "startupProbe: Dá tempo para iniciar. livenessProbe: Detecta se travou. readinessProbe: Indica se pode receber tráfego. Juntas, garantem alta disponibilidade."
          - type: "tip"
            title: "kubectl rollout status"
            content: "Este comando acompanha o progresso do rollout em tempo real e só retorna quando o deploy está completo ou falhou."
        validation:
          - command: "kubectl get deployment meu-servico -n producao -o jsonpath='{.status.readyReplicas}' 2>/dev/null"
            expectedOutput: "3"
            errorMessage: "O Deployment corrigido não está com 3 réplicas prontas. Verifique se aplicou o manifesto corrigido."
          - command: "kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].readinessProbe.httpGet.path}' 2>/dev/null"
            expectedOutput: "/"
            errorMessage: "A readinessProbe não está configurada corretamente."

      - name: "Lições Aprendidas: O Post-Mortem"
        description: "Na segunda-feira, na reunião de time, você apresenta o post-mortem. O susto de sexta virou lição para todos."
        steps:
          - "**Segunda-feira, Reunião de Time**"
          - "Aquele deploy de sexta-feira se tornou uma lição valiosa. Você apresenta o post-mortem."
          - "Vamos documentar as lições. Crie o arquivo `POST-MORTEM.md`:"
          - |
            ```markdown
            # Post-Mortem: O Deploy de Sexta-Feira

            ## Resumo do Incidente
            - **Data:** Sexta-feira, 16h
            - **Duração:** ~2 horas
            - **Impacto:** Serviço parcialmente indisponível
            - **Causa Raiz:** Dívida técnica (falta de resources e probes)

            ## O que aconteceu?
            Uma alteração "simples" de código aumentou levemente o consumo de memória,
            causando OOMKill em Pods que não tinham limites adequados nem probes
            configuradas.

            ## Por que aconteceu?
            1. **Falta de Gestão de Recursos (Day-2)**
               - Sem requests e limits, o Pod era uma bomba-relógio
               - Qualquer variação no consumo → OOMKilled

            2. **Estratégia de Rollout Otimista (Day-3)**
               - RollingUpdate sem readiness probes
               - Kubernetes não sabia se o novo Pod estava pronto

            3. **Ausência de Verificações de Saúde (Day-4)**
               - Sem Readiness Probe → tráfego para Pods quebrados
               - Sem Liveness Probe → Pods "zumbis" no ar

            ## Ações Corretivas
            - [x] Adicionar resources (requests/limits) em todos os Deployments
            - [x] Implementar startupProbe, livenessProbe e readinessProbe
            - [ ] Implementar políticas com OPA/Kyverno para bloquear deploys sem probes
            - [ ] Adicionar alarmes para OOMKill no cluster

            ## Lições Aprendidas
            1. Recursos não são opcionais - são a BASE da estabilidade
            2. Deploy seguro = estratégia + probes trabalhando juntos
            3. Sua aplicação precisa "conversar" com o Kubernetes via probes
            ```
          - "Agora, vamos verificar que nosso sistema está robusto. Confira os recursos:"
          - "`kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].resources}' | jq .`"
          - "Confira as probes:"
          - "`kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0]}' | jq '{startupProbe, livenessProbe, readinessProbe}'`"
          - "**🎓 O que começou como um susto de sexta-feira terminou com um sistema mais robusto e um time mais experiente.**"
          - "E essa é uma das melhores partes de trabalhar com tecnologia: **Problema resolvido!**"
        tips:
          - type: "success"
            title: "🏆 Parabéns, Detetive!"
            content: "Você completou a investigação! Agora você sabe diagnosticar e corrigir problemas de OOMKill, Probes e Rolling Updates no Kubernetes."
          - type: "info"
            title: "Post-Mortems São Essenciais"
            content: "Documentar incidentes sem culpar pessoas (blameless) ajuda todo o time a aprender. O objetivo é melhorar processos, não apontar dedos."
          - type: "tip"
            title: "Próximos Passos"
            content: "Implemente políticas de admissão (OPA Gatekeeper ou Kyverno) para bloquear Deployments sem resources e probes em produção. Prevenção > Correção!"
        validation:
          - command: "kubectl get pods -n producao -l app=meu-servico --no-headers 2>/dev/null | grep -c 'Running' || echo '0'"
            expectedOutput: "3"
            errorMessage: "Nem todos os pods estão Running. Verifique o status do Deployment."
          - command: "kubectl get deployment meu-servico -n producao -o jsonpath='{.spec.template.spec.containers[0].livenessProbe.httpGet.path}' 2>/dev/null"
            expectedOutput: "/"
            errorMessage: "A livenessProbe não está configurada corretamente no Deployment final."
```