## Deployment
No Kubernetes Deployments estão em um nível acima de abstração que os replicasets. Eles são capazes de modificar as imagens dos pods de maneira segura, tambpem podendo realizar o rollback dos pods. Realizando alterações nos ReplicaSets e Pods.  
```sh
kubectl get deployments
```

*DICA*: Use o comando kubectl run para gerar templates do agente que deseja. Veja os exemplos:  
```sh
kubectl run nginx --image=nginx   #cria um pod com imagem NGINX
kubectl run nginx --image=nginx --dry-run=client -o yaml   #cria o template YAML do pod mas não cria ele no cluster
kubectl create deployment --image=nginx nginx   #cria um deploy 
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml   #cria um template YAML do deploy mas não cria ele no cluster 
kubectl create deployment --image=nginx nginx --dry-run=client -o yaml > nginx-deploy-definition.yaml   #escreve o template em um arquivo
```
