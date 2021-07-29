# Rollout e Rollback 
Existem duas estrategias de Deploy no K8s, `Recreate` e `RollingUpdate`.  
- Recreate: escala as replicas antigas para zero e depois cria novas replicas 
- RollingUpdate: escala as novas replicas enquanto as antigas continuam rodando, depois que as novas replicas estiverem bem, ele as remove.
   
Podemos fazer o rollback de um deploy pelo seguinte comando:  
```sh
kubectl rollout undo deployment/<nome-deploy>
``` 
  
Podemos ver o status de cada deploy usando o seguinte comando:  
```sh
kubectl rollout status deployment/<nome-deploy>
kubectl rollout history deployment/<nome-deploy>
```  
  
Para pode fazer atualizacoes na imagem do deploy podemos usar:  
```sh
kubectl apply -f manifesto-deploy.md
kubectl set image deployment/<nome-deploy> <nome-container>=<nome-imagem>
```  
Podemos atualizar a versao de um deployment aplicando o manifesto atualizado novamente, assim ele sera atualizado com as novas configuracoes. 
