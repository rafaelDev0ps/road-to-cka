# Problemas no Controlplane
Quando acontece falha no Node master (Controplane) o que devemos fazer para debuggar?  
  
- Checar status dos Nodes do cluster
- Verificar status/eventos dos Pods no namespace `kube-system`
- Verificar se os services (daemons) como `kube-apiserver`, `kube-controller-manager`, `kube-scheduler` estao funcionando 
- Verificar se os services (deamons) no Worker Node como `kubelet` e `kube-proxy` estao funcionando
- Verifique os logs do controlplane com `kubectl logs kube-apiserver-master -n kube-system` ou `sudo journalctl -u kube-apiserver`  
  
Caso nao encontre o erro, veja mais na documentacao de [Troubleshooting Clusters](https://kubernetes.io/docs/tasks/debug-application-cluster/debug-cluster/)  
  
**OBSERVACAO**
Para ver o status de um daemon use `service nome-service status`. Por exemplo, `service kubelet status`.  
  
***Vale lembrar que os Pods do Controlplane que ficam no namespace `kube-system` sao Pods estaticos que sao configurados em `/etc/kubernetes/manifests/`*