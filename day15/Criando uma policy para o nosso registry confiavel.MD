### Exemplo de Política: Permitir Apenas Repositórios Confiáveis

A política ensure-images-from-trusted-repo é essencial para a segurança dos clusters Kubernetes, garantindo que todos os Pods utilizem imagens provenientes apenas de repositórios confiáveis. Esta política ajuda a prevenir a execução de imagens não verificadas ou potencialmente mal-intencionadas.

**Detalhes da Política**

Esta política impõe que todas as imagens de containers usadas nos Pods devem ser originárias de repositórios especificados e confiáveis. A estratégia é crucial para manter a integridade e a segurança do ambiente de containers, evitando riscos associados a imagens desconhecidas ou não autorizadas.

**Arquivo de Política: registry-allowed.yaml**

```yaml
apiVersion: kyverno.io/v1
kind: ClusterPolicy
metadata:
  name: ensure-images-from-trusted-repo
spec:
  validationFailureAction: enforce
  rules:
  - name: trusted-repo-check
    match:
      resources:
        kinds:
        - Pod
    validate:
      message: "Only images from trusted repositories are allowed"
      pattern:
        spec:
          containers:
          - name: "*"
            image: "trustedrepo.com/*"
```

**Implementação e Impacto**

Com a implementação desta política, qualquer tentativa de implantar um Pod com uma imagem de um repositório não confiável será bloqueada. A política assegura que apenas imagens de fontes aprovadas sejam usadas, fortalecendo a segurança do cluster contra vulnerabilidades e ataques externos.

