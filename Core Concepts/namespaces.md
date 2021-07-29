## Namespaces
Corresponde a um grupo com recursos (Pods, Deploy, Services e entre outros). Por padrão quando o cluster é criado o namespace default é criado também.  
Namespaces servem para organizar melhor o cluster, criando namespaces para produção, desenvolvimento, holomogações e etc.  
  
O formato padrão de um DNS em um namespace é `<nome-service>.<namespace>.<service>.<domain>`. 
Por exemplo: `db-service.dev.svc.cluster.local`  
```sh
kubectl create namespace dev
kubectl create ns dev
```
  
Context servem para diferenciar clusters diferentes.  
```sh
kubectl get pods -n dev --context cluster1
```
