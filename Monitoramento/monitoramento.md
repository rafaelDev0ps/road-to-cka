# Monitoramento

Podemos coletar informacoes dos Nodes e Pods atraves dos seguintes comandos.  
```sh
kubectl top node
kubectl top node <nome-node>
kubectl top po <nome-pod>
kubectk logs <nome-pod> -c <nome-container>
```
  
Se a API de monitoramento nao estiver instalada no Node desejado, basta instalar assim:  
```sh
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```  
Para mais informacoes veja (aqui)[https://github.com/kubernetes-sigs/metrics-server]
  
Para visualizar os logs de um pod em tempo real podemos fazer assim:  
```sh
kubectl logs <nome-pod> -f
kubectl logs <nome-pod> -f -c <nome-container>
```  
