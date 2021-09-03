# Seguranca das imagens
As imagens que sao baixadas nos Pods seguem a seguinte convecao:  
```yaml
image: nome-user-registry/nome-imagem
```  
Caso voce coloque somente o nome da imagem, o K8s vai entender que o nome do user do registry sera o mesmo.  
  
_Vale lembrar que Registry eh o repositorio onde as imagens ficam salvas para poderem ser encontradas!_  
  
O registry padrao eh o DockerHub `docker.io`. Logo o nome inteiro da imagem fica:  
```yaml
image: docker.io/nome-user-registry/nome-imagem
```  
  
## Registries Privados
Quando eh usado um registry privado precisamos configurar as credenciais para que o K8s possa acessar e fazer o pull das imagens.  
```sh
kubectl create secret docker-registry credenciais-registry \
    --docker-server=meu-server-de-imagens.io \
    --docker-username=meu-username \
    --docker-password=minha-senha \
    --docker-email=meu@email.com
```  
  
Depois disso devemos  nos Pods a Secret que deve ser usada para fazer o pull das imagens:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: meu-pod
spec:
  containers:
  - name: meu-app
    image: meu-server-de-imagens.io/usuario/meu-app
  imagePullSecrets:  # <=== AQUI
  - name: credenciais-registry  # <=== AQUI
```  
