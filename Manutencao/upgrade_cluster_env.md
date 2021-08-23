## Atualização do SO do cluster
No Kubernetes, se um node ficar 5 minutos fora do ar ele é considerado "morto" e com isso os Pods que executam nele são terminados. Esse comportamento pode ser configurado passando um comando para o controller:  
```sh
kube-controller-manager --pod-eviction-timeout=10m0s
```  
Dessa maneira se o Node ficar mais de 10 minutos fora do ar ele será considerado morto.  
Quando um Node volta a funcionar após ficar fora do ar ele é inicializado sem nenhum Pod instalado.  
  
*Se você for realizar um update rapido no Node dentro desse periodo de downtime as configurações do cluster não serão perdidas.*  
  
Porém existe uma maneira mais segura de realizar um update no cluster. Podemos realizar um `drain` no Node que será atualizado e com isso ele irá mover os Pod para outros Nodes disponíveis.  
```sh
kubectl drain <nome-node-atualizado>
```  
Nesse caso os Pods serão terminados no Node que sofrerá atualização e serão recriados em outro Node. Além disso, o Node a ser atualizado será marcado como `unschedulable` ou `cordoned` fazendo com que não seja possível subir nenhum Pod nele.  
Assim que o Node for atualizado e voltar a funcionar é necessário habilitar novamente o scheduling dele para possamos instalar os Pods novamente. Para isso fazemos:  
```sh
kubectl uncordon <nome-node-atualizado>
```  
Os Pods que foram movidos para outros Nodes não irão voltar automaticamente para o Node de origem, para que isso aconteça é necessário deletar esses Pods para que consigam ser instalados no Node de origem.  
  
Como comentado anteriormente podemos marcar como `cordon` para que o cluster não autorize instalar Pods nele. Para isso podemos fazer:  
```sh
kubectl cordon <nome-node>
``` 
Diferente do comando drain ele não irá mover os Pods instalados no Node para outro Node. Apenas irá garantir que novos Pods NÃO serão instalados nele. 
  
### IMPORTANTE
Não é possivel fazer o drain de um Node que contém Pods que não fazem parte de um ReplicaSet, Job, DaemonSet ou StatefulSet. Para que isso seja possível, é necessário aplicar a flag `--force`. Com isso, se o drain for forçado, os Pods que não são gerenciados por um dos agentes mencionados será deletado para sempre. 