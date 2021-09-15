# Service Networking
Quando temos que comunicar um Pod com outro dentro do cluster, precisamos configurar um Service. Existem alguns tipos de Service e veremos em seguinda.  
  
Quando Services sao criados no cluster eles sao acessives por todos os Pods do cluster. Mesmo que um Service seja criado em um Node especifico ele nao se limita somente para esse Node.

## Service ClusterIP
Esse tipo de Service acessivel para todos os Pods do cluster se chama ClusterIP. Por exemplo, caso tenhamos um banco de dados instalado no cluster e que ele seja acessivel pelo cluster todo devemos aplicar esse service para que ele se comunique com os demais Pods.  
  
## Service NodePort
Quando temos uma aplicacao web que precisa se comunicar com agentes externos do cluster, o ideal eh que seja usado um Service do tipo NodePort.  
Com o NodePort ele estabelece um IP para o Service criado (assim como no ClusterIP) porem ele tambem abre uma porta especifica dos Nodes do cluster para que chamadas externas sejam feitas nessa porta.  
  
**_Quem eh o responsavel por criar esses Services e pelo funcionamento deles?_**  
O agente responsavel por isso eh o `kube-proxy`, com ele cuida dos Services. Como Services nao sao aplicacoes propriamente ditas como os Pods, eles devem estar registrados em todo o cluster. Services sao objetos virtuais assim como os Network Namespaces e interfaces virtuais.  
  
### kube-proxy
O `kube-proxy` eh um conjunto de Pods gerenciados por DaemonSets e esses Pods existem em cada Node do cluster.  
Quando um service eh criado no cluster o `kube-proxy` cria regras de forwarding para que haja um redirecionamento para o IP e porta do service.  
Por padrao, o `kube-proxy` faz isso atraves do `iptables`. Podemos confirmar ferramenta que o `kube-proxy` usa verificando os logs do Pod.  
```sh
$ kubectl logs kube-proxy-q4n6m -n kube-system  

...
I0915 12:33:33.030219       1 server_others.go:185] Using iptables Proxier.
...
```  
De fato esta utilizando iptables chamado `Proxier`.  
  
Eh atraves disso que ele define um IP para um service criado. Podemos ver isso da seguinte maneira:  
```sh
$ ps aux | grep kube-apiserver
```  
Voce ira ver que uma das configuracoes do `kube-apiserver` eh `--service-cluster-ip-range=10.96.0.0/12`. Isso significa o range de IP definido para os Service do tipo ClusterIP (e tambem NodePort) irao respeitar `10.96.0.0/12`, com isso havera IPs de `10.96.0.1` ate `10.111.255.254`.  
  
Podemos ver essas regras de redirecionamento da seguinte forma:  
```sh
$ iptables -L -t nat | grep meu-service

...
target     prot opt source               destination         
KUBE-SVC-SZZ7MOWKTWUFXIJT  tcp  --  anywhere             10.33.9.4             /* meu-namespace/meu-service: cluster IP */ tcp dpt:27017
DNAT                       tcp  --  anywhere             anywhere              /* meu-namespace/meu-service: */ tcp to:10.118.2.3:27017
KUBE-SEP-3DU66DE6VORVEQVD  all  --  anywhere             anywhere              /* meu-namespace/meu-service*/
...
```  
No log acima vemos que o IP do Service `meu-service` eh `10.33.9.4` e sera redirecionado para o IP do Pod que eh `10.118.2.3` na porta `27017`.  

Eh possivel ver atraves dos logs do `kube-proxy` atraves do arquivo de logs `/var/log/kube-proxy.log`  
  
