## Forma Imperativa vs Forma Declarativa
Na forma imperativa os passos sao executados sequencialmente pelo usuario, enquanto na forma declarativa o usuario descreve os passos para serem executados pelo computador sequencialmente.  
  
Exemplos de comandos Imperativos do Kubernetes: 
```sh
kubectl run ubuntu --image=ubuntu
kubectl create deployment --image=redis
kubectl edit ...
kubectl scale ...
kubectl set ...
kubectl replace ...
kubectl delete ...
```
  
Ja nos exemplos declarativos do K8s usa-se arquivos YAML para que o kube-apiserver valideas configuracoes solicitadas.  
  
Formas para criar e editar configuracoes:  
```sh
kubectl apply -f <nome-arquivo>.yaml
```
