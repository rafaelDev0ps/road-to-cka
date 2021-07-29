## Replication Controller != ReplicaSet
Replication Controller cria multiplas instancias de um pod no node.  
```sh
kubectl get replicationcontroller
```  
  
Replicaset depende de um selector para saber quais pods são desejados, essa é a principal diferença (no template) entre replication controller e replicaset.  
Replicaset é um processo responsável por monitorar os pods.  
```sh
kubectl get replicaset
```  

_Porque colocamos labels e selector nos pods?_  
- Para facilitar a detecção de pods para serem manipulados por outros agentes Kubernetes.  
  
_Como escalar mais replicas de um pod?_  
- Editar o template de replicaset
- `kubectl scale --replicas=6 replica_set_template.yaml`
- `kubectl scale --replicas=6 replicaset my-replicaset`
  
### IMPORTANTE
- Prestar bastante atenção nos labels usados no template, se estiverem diferentes as replicas não serão criadas corretamente.
- Utilize o comando kubectl explain replicaset para ver a documentação do replicaset no console.
