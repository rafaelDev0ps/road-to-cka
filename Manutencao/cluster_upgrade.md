# Como atualizar um cluster K8s
O Kubernetes é formado por um conjunto de APIs, cada API pode ter uma versão diferente da outra contanto que seja uma versão *menor ou igual* a da versão do `kube-api`.  
Logo o Controller Manager e o `kube-scheduler` podem ser uma versão menor que a do `kube-api`.  
E o `kubelet` e o `kube-proxy` podem ser versões menores que o `Controller Manager` e o `kube-scheduler`.  
  
_Ok, mas quando eu devo fazer o upgrade do meu cluster?_  
No Kubernetes, é dado suporte para as três últimas versões. Portanto se a versão do seu cluster não estiver dentro desse range, você deve atualizar assim que possível para que tenha o suporte necessário.  
  
_Ok, mas como eu atualizo o cluster?_  
O recomendado é atualizar uma Minor Release de cada vez. Logo, se você está na versão `1.18.1` o recomendado é que atualize para a versão `1.19.1` e assim por diante.  
O processo irá depender como o seu cluster está instalado, se você estiver usando um provedor Cloud, você pode usar o provedor como um meio de atualização automática para você.  
Caso você tenha usado o `kubeadm` para instalar o cluster, os comandos abaixo são usados:  
```sh
kubectl upgrade plan
kubectl upgrade apply
```  
Se você instalou o cluster da maneira masoquista (na mão), você pode atualizar cada componente separadamente.  
  
## Usando o Kube ADM
Existem dois principais passos para atualizar seu cluster K8s.
1. Atualizar o Master Node
2. Atualizar o Worker Node
  
Quando você atualizar seu Master Node ele ficará indisponível por uns instantes, mas isso não significa que seus Worker Nodes ficarão indisponíveis também. Porém, como o Master está caído você não consegue fazer o gerenciamento do seu cluster, não acessá-lo.  
  
_E os Worker Nodes?_  
Existem tres maneiras de atualizar os Workers.
- Você pode atualizar todos de uma vez, porém isso vai deixar todas as suas aplicações inacessíveis. Assim que forem atualizados tudo volta ao normal.  
- Você pode atualizar um Worker de cada vez, movendo os agentes instalados para os outros Workers enquando esse está sendo atualizado. E assim acontece com cada Node sem haver downtime.  
- Voce pode adicionar um novo Node no seu cluster com a versao atualizada, assim voce pode mover o workload do Node desatualizado para o novo Node e depois remove-lo.
  
_Como essas atualizacoes sao feitas?_
Para saber qual a versao LTS que deve ser atualizado execute o comando:  
```sh
kubeadm upgrade plan
```  
Atraves do kubeadm com o comando:  
```sh
kubeadm upgrade apply v<nova-versao>
```  
  
Com isso o `kubeadm` ira fazer o upgrade dele mesmo e estara preparado para atualizar os componentes do cluster.  
*Note que se voce tentar ver os Nodes com* `kubectl get nodes` *vera que o master Node ainda esta na versao antiga*. Isso acontece porque o `kubectl` esta mostrando a versao do `kubelet` em cada Node ao inves da versao dos componentes de cada Node.    
### Ordem de instalacao
1. Master Node
2. Worker Nodes
  
### Update Master Node
Dentro do Master Node execute:
```sh
apt-get update
apt-get install kubeadm=1.20.0-00
kubeadm version
kubeadm upgrade plan
kubeadm upgrade apply v1.20.0-00
```
Agora que voce atualizou o kubeadm, atualize o kubelet:  
```sh
kubectl drain <nome-master-node> --ignore-daemonsets
apt-get update
apt-get install kubelet=1.20.0-00
systemctl restart deamon-reload
systemctl restart kubelet
kubectl uncordon <nome-node-master>
```

### Update Worker Node
Dentro do seu Worker Node execute:  
```sh
apt-get update
apt-get install kubeadm=1.20.0-00
kubeadm upgrade node
```
Agora dentro do seu MASTER NODE execute:  
```sh 
kubectl drain <nome-worker-node> --ignore-daemonsets
```  
Voltando para dentro do Worker Node, execute:  
```sh
apt-get update
apt-get install kubelet=1.20.0-00
systemctl restart daemon-reload
systemctl restart kubelet
```  
Volte para o Master Node e reative o Worker Node:
```sh
kubectl uncordon <nome-worker-node>
```  
  
## Importante
O `kubeadm` nao instala nem atualiza o `kubelet`. Eles devem ser atualizados separadamente.  
Vale lembrar novamente que quando um Node recebe `uncordon` e necessario deletar os Pods que foram movidos para que eles voltem para o Node de origem. 