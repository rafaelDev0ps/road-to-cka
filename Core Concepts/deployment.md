# Deployment
No Kubernetes Deployments estão em um nível acima de abstração que os replicasets. Eles são capazes de modificar as imagens dos pods de maneira segura, tambem podendo realizar o rollback dos pods. Realizando alterações nos ReplicaSets e Pods.  
  
Podemos criar um deployment a partir do manifesto:  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:           ## DADOS PARA CONSEGUIR BUSCAR ESSE DEPLOY
  name: nginx-deployment
  labels:
    app: nginx
spec:
  replicas: 3  # NUMERO DE REPLICAS DESEJADO
  selector:     # Selector serve para atribuir esse deploy em Policies, Services e outros agentes
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:           ## SPECS DOS CONTAINERES
      containers:
      - name: nginx
        image: nginx:1.14.2
        ports:
        - containerPort: 80
```  
  
Para buscar um deploy ou lista-los:
```sh
kubectl get deployments
kubectl get deploy 
kubectl get deploy <nome-deploy>
```  
O nome do deploy eh atribuido no campo `metadata.name` no manifesto.  
  
Podemos buscar o deploy ou qualquer agente atraves dos labels:
```sh
kubectl get deploy -l app=nginx
```  
Com a flag -l podemos buscar os agentes pelo `metadata.labels` que esta no manifesto. Acima temos um exemplo onde busca os deployments com o label `app: nginx`.
  
## Outras operacoes
Com o deploy voce pode realizar outras operacoes como `scale`, `edit`, `describe`, `delete` e `rollout`.  

## Estrategias de deploy  
Alem disso, existem duas estrategias de deploy.
- `RollingUpdate`: mata uma parcela dos Pods antigos e sobe uma parcela dos Pods novos e gradativamente vai atualizando os Pods. Essa parcela pode ser configurada atraves do campo `.spec.strategy.rollingUpdate.maxUnavailable` e `.spec.strategy.rollingUpdate.maxSurge`.
- `Recreate`: mata todos os Pods antigos primeiro e sobe os Pods novos
  
**DICA**: Use o comando kubectl run para gerar templates do agente que deseja. Veja os exemplos:  
```sh
kubectl run nginx --image=nginx   #cria um pod com imagem NGINX
kubectl run nginx --image=nginx --dry-run=client -o yaml   #cria o template YAML do pod mas não cria ele no cluster
kubectl create deployment --image=nginx nginx   #cria um deploy 
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml   #cria um template YAML do deploy mas não cria ele no cluster 
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deploy-definition.yaml   #escreve o template em um arquivo
```
