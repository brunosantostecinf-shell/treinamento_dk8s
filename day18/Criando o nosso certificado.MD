### Criando um Usuário para acesso ao cluster

Bem, agora que já sabemos quais serão as permissões do nosso novo usuário, já podemos iniciar a sua criação.

Primeira coisa que precisamos é criar uma chave privada para o nosso usuário. Para isso, vamos utilizar o comando openssl:

```bash
openssl genrsa -out developer.key 2048
```

Com o comando acima estamos criando uma chave privadas de 2048 bits e salvando ela no arquivo developer.key. O parametro genrsa indica que queremos gerar uma chave privada, e o parametro -out indica o nome do arquivo que queremos salvar a chave.

Com a chave criada, precisa agora criar a um CSR, ou Certificate Signing Request, que é um arquivo que contém o certificado que criamos, e que será enviado para o Kubernetes assinar e gerar o certificado final.

```bash
openssl req -new -key developer.key -out developer.csr -subj "/CN=developer"
```

No comando acima estamos criando um certificado para o nosso usuário, utilizando a chave privada que criamos anteriormente. O parametro req indica que queremos criar um certificado, o parametro -key indica o nome do arquivo da chave privada que queremos utilizar, o parametro -out indica o nome do arquivo que queremos salvar o certificado, e o parametro -subj indica o nome do usuário que queremos criar.

Pronto, agora com os dois arquivos em mãos, já podemos iniciar a criação do nosso usuário no cluster, mas antes, precisamos criar um CSR, ou Certificate Signing Request, que é um arquivo que contém o certificado que criamos, e que será enviado para o Kubernetes assinar e gerar o certificado final.

Mas para que possamos criar o arquivo, precisamos antes ter o conteúdo do certificado em base64, para isso, vamos utilizar o comando base64:

```bash
cat developer.csr | base64 | tr -d '\n'
```

Com o comando acima estamos lendo o conteúdo do arquivo developer.csr, convertendo ele para base64, e removendo a quebra de linha.

O conteúdo do certificado em base64 será algo parecido com isso:

```
LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0ZERVNNQkFHQTFVRUF3d0paR1YyWld4dmNHVnlNSUlCSWpBTkJna3Foa2lHOXcwQgpBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF5OFN1Y25JWjJjL0k3dXQvS0EwSXhvN0RZa0hkSUxrbmxZOWNwMkVlClJJSzRzU1NzZnA2SzBhbWZlYWtRaEdXT2NWMmFaeEtTM0xrNERNNlVmb3ZucEQvOXpidDl0em44UUpMTDZxREEKeHFxbzVSbEt4QnpEV3lQT2JkUStMWnI2VjFQZ2wxYms2c3o0d2lWek52a2NhT0doSDdlSU90QVI0U096MjNJdAowZ0xiZHBDalFITFIvNlFuSXBjY3h3bDBGa1FtL3RVeHdRa0x1NXNpSTNKOGRiUkQwcnlFdGxReWQ5elhLM29rCjBRbVpLZDVpV1p2aDU3R1lrV1kweGMzV0J5aXY5OURQYVE3WTB4MFNaWGlPL2w0bTRzazJ3RjYwa2dUa1NJZmQKdEMxN2Y1ZzVWVzhOTW02amNpMFRXeDk5Z0REcmpHanJpaExHeTBLUWdRa2p3d0lEQVFBQm9BQXdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUllZVdLbjAwZkk5ekw3MUVQNFNpOUVxVUFNUnBYT0dkT1Aybm8rMTV2VzJ5WmpwCmhsTWpVTjMraVZubkh2QVBvWFVLKzBYdXdmaklHMjBQTjA5UEd1alU4aUFIeVZRbFZRS0VaOWpRcENXYnQybngKVlZhYUw0N0tWMUpXMnF3M1YybmNVNkhlNHdtQzVqUE9vU29vVGtrVlF5Uml4bkcyVVQrejI3M2xpaTY3RkFXegpBZ1QvczlVa3gvS1dxRjIzczVuUk9TTlZUS2xCSG5LMU40YkN6RHBqZnN5V01GUXdnazhxRCtlOXp0cTh2c1VhCi9Say9jUWNyS2wxVDMyM0xDcG1TekhnM3hDdjFqdzJUVFFINm1yWlBBa2doa2R2YlNnalp6Y1JRZWNqSEpNeTMKTzFJQXJ6V3pWbU1hRTJqeGhUV1JwbkJkcVZjZERTUERiNkNXaktVPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K%
```

Lembre-se de remover a quebra de linha do final do arquivo, representada pelo %.

Agora que já temos o conteúdo do certificado em base64, copie ele e cole no arquivo developer.yaml que vamos criar agora:

```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
 name: developer
spec:
 request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0ZERVNNQkFHQTFVRUF3d0paR1YyWld4dmNHVnlNSUlCSWpBTkJna3Foa2lHOXcwQgpBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF5OFN1Y25JWjJjL0k3dXQvS0EwSXhvN0RZa0hkSUxrbmxZOWNwMkVlClJJSzRzU1NzZnA2SzBhbWZlYWtRaEdXT2NWMmFaeEtTM0xrNERNNlVmb3ZucEQvOXpidDl0em44UUpMTDZxREEKeHFxbzVSbEt4QnpEV3lQT2JkUStMWnI2VjFQZ2wxYms2c3o0d2lWek52a2NhT0doSDdlSU90QVI0U096MjNJdAowZ0xiZHBDalFITFIvNlFuSXBjY3h3bDBGa1FtL3RVeHdRa0x1NXNpSTNKOGRiUkQwcnlFdGxReWQ5elhLM29rCjBRbVpLZDVpV1p2aDU3R1lrV1kweGMzV0J5aXY5OURQYVE3WTB4MFNaWGlPL2w0bTRzazJ3RjYwa2dUa1NJZmQKdEMxN2Y1ZzVWVzhOTW02amNpMFRXeDk5Z0REcmpHanJpaExHeTBLUWdRa2p3d0lEQVFBQm9BQXdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUllZVdLbjAwZkk5ekw3MUVQNFNpOUVxVUFNUnBYT0dkT1Aybm8rMTV2VzJ5WmpwCmhsTWpVTjMraVZubkh2QVBvWFVLKzBYdXdmaklHMjBQTjA5UEd1alU4aUFIeVZRbFZRS0VaOWpRcENXYnQybngKVlZhYUw0N0tWMUpXMnF3M1YybmNVNkhlNHdtQzVqUE9vU29vVGtrVlF5Uml4bkcyVVQrejI3M2xpaTY3RkFXegpBZ1QvczlVa3gvS1dxRjIzczVuUk9TTlZUS2xCSG5LMU40YkN6RHBqZnN5V01GUXdnazhxRCtlOXp0cTh2c1VhCi9Say9jUWNyS2wxVDMyM0xDcG1TekhnM3hDdjFqdzJUVFFINm1yWlBBa2doa2R2YlNnalp6Y1JRZWNqSEpNeTMKTzFJQXJ6V3pWbU1hRTJqeGhUV1JwbkJkcVZjZERTUERiNkNXaktVPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K 
 signerName: kubernetes.io/kube-apiserver-client
 expirationSeconds: 31536000 # 1 year
 usages:
 - client auth
```
No arquivo acima, estamos definindo as seguintes informações:

* apiVersion: Versão da API que estamos utilizando para criar o nosso usuário.
* kind: Tipo do recurso que estamos criando, no caso, um CSR.
* metadata.name: Nome do nosso usuário.
* spec.request: Conteúdo do certificado em base64.
* spec.signerName: Nome do assinador do certificado, que no caso é o kube-apiserver, que será o responsável por assinar o nosso certificado.
* spec.expirationSeconds: Tempo de expiração do certificado, que no caso é de 1 ano.
* spec.usages: Tipo de uso do certificado, que no caso é client auth.

Agora que já temos o nosso arquivo criado, vamos aplicar ele no cluster:

```
kubectl apply -f developer.yaml
```

Vamos listar os CSR's do cluster para ver o status do nosso usuário:

```
kubectl get csr
```

O resultado será algo parecido com isso:

```
NAME        AGE   SIGNERNAME                                    REQUESTOR                 REQUESTEDDURATION   CONDITION
csr-4zd8k   15m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-68wsv   15m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-jkm8t   15m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-r2hcr   15m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-x52kj   15m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
```

developer   3s    kubernetes.io/kube-apiserver-client           kubernetes-admin          365d                Pending
Perceba que o nosso usuário está com o status Pending, isso porque o kube-apiserver ainda não assinou o nosso certificado. Você pode acompanhar o status do seu usuário através do comando:

```
kubectl describe csr developer
```
O resultado será algo parecido com isso:

```
Name:         developer
Labels:       <none>
Annotations:  kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"certificates.k8s.io/v1","kind":"CertificateSigningRequest","metadata":{"annotations":{},"name":"developer"},"spec":{"expirationSeconds":31536000,"request":"LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1dUQ0NBVUVDQVFBd0ZERVNNQkFHQTFVRUF3d0paR1YyWld4dmNHVnlNSUlCSWpBTkJna3Foa2lHOXcwQgpBUUVGQUFPQ0FROEFNSUlCQ2dLQ0FRRUF5OFN1Y25JWjJjL0k3dXQvS0EwSXhvN0RZa0hkSUxrbmxZOWNwMkVlClJJSzRzU1NzZnA2SzBhbWZlYWtRaEdXT2NWMmFaeEtTM0xrNERNNlVmb3ZucEQvOXpidDl0em44UUpMTDZxREEKeHFxbzVSbEt4QnpEV3lQT2JkUStMWnI2VjFQZ2wxYms2c3o0d2lWek52a2NhT0doSDdlSU90QVI0U096MjNJdAowZ0xiZHBDalFITFIvNlFuSXBjY3h3bDBGa1FtL3RVeHdRa0x1NXNpSTNKOGRiUkQwcnlFdGxReWQ5elhLM29rCjBRbVpLZDVpV1p2aDU3R1lrV1kweGMzV0J5aXY5OURQYVE3WTB4MFNaWGlPL2w0bTRzazJ3RjYwa2dUa1NJZmQKdEMxN2Y1ZzVWVzhOTW02amNpMFRXeDk5Z0REcmpHanJpaExHeTBLUWdRa2p3d0lEQVFBQm9BQXdEUVlKS29aSQpodmNOQVFFTEJRQURnZ0VCQUllZVdLbjAwZkk5ekw3MUVQNFNpOUVxVUFNUnBYT0dkT1Aybm8rMTV2VzJ5WmpwCmhsTWpVTjMraVZubkh2QVBvWFVLKzBYdXdmaklHMjBQTjA5UEd1alU4aUFIeVZRbFZRS0VaOWpRcENXYnQybngKVlZhYUw0N0tWMUpXMnF3M1YybmNVNkhlNHdtQzVqUE9vU29vVGtrVlF5Uml4bkcyVVQrejI3M2xpaTY3RkFXegpBZ1QvczlVa3gvS1dxRjIzczVuUk9TTlZUS2xCSG5LMU40YkN6RHBqZnN5V01GUXdnazhxRCtlOXp0cTh2c1VhCi9Say9jUWNyS2wxVDMyM0xDcG1TekhnM3hDdjFqdzJUVFFINm1yWlBBa2doa2R2YlNnalp6Y1JRZWNqSEpNeTMKTzFJQXJ6V3pWbU1hRTJqeGhUV1JwbkJkcVZjZERTUERiNkNXaktVPQotLS0tLUVORCBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0K","signerName":"kubernetes.io/kube-apiserver-client","usages":["client auth"]}}

CreationTimestamp:   Wed, 31 Jan 2024 11:52:24 +0100
Requesting User:     kubernetes-admin
Signer:              kubernetes.io/kube-apiserver-client
Requested Duration:  365d
Status:              Pending
Subject:
         Common Name:    developer
         Serial Number:  
Events:  <none>
```

Tudo certo até aqui, agora precisamos assinar o nosso certificado, para isso, vamos utilizar o comando kubectl certificate approve:

```
kubectl certificate approve developer
```

Agora vamos listar os CSR's do cluster novamente:

```
kubectl get csr
```

O resultado será algo parecido com isso:

```
NAME        AGE   SIGNERNAME                                    REQUESTOR                 REQUESTEDDURATION   CONDITION
csr-4zd8k   17m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-68wsv   17m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-jkm8t   17m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-r2hcr   17m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
csr-x52kj   16m   kubernetes.io/kube-apiserver-client-kubelet   system:bootstrap:abcdef   <none>              Approved,Issued
developer   88s   kubernetes.io/kube-apiserver-client           kubernetes-admin          365d                Approved,Issued
```

Pronto, o nosso certificado foi assinado com sucesso, agora podemos pegar o certificado do nosso usuário e salvar em um arquivo, para isso, vamos utilizar o comando kubectl get csr:

```
kubectl get csr developer -o jsonpath='{.status.certificate}' | base64 --decode > developer.crt
```

No comando acima, estamos pegando o certificado do nosso usuário, convertendo ele para base64, e salvando ele no arquivo developer.crt.

Para pegar o certificado, estamos usando o parametro -o jsonpath='{.status.certificate}', para que o comando retorne apenas o certificado do usuário, e não todas as informações do CSR.

Você pode conferir o conteúdo do certificado através do comando:

```
cat developer.crt
```

Pronto, agora temos o nosso certificado final criado, e podemos utilizá-lo para acessar o cluster, mas antes precisamos definir o que o nosso usuário pode fazer no cluster.

