# Releases e versões do Kubernetes
Podemos ver a versão do Node através do comando  
```sh
kubectl get nodes
```  
Além disso, existem outros componentes no Kubernetes que possuem suas próprias versões como o ETCD CLuster e o CoreDNS.  
A forma como o K8s é versionado é bem simples. Existem três partes:  
  
Exemplo: v1.20.1  
A primeira parte é chamada de Major Release, nela são lançadas versões estaveis do Kubernetes.  
A segunda parte é chamada de Minor Release, onde são inseridas features e funcionalidades da ferramenta.  
A terceira parte é chamada de Patch, nela é lançada as correções de bugs e hotfixes.