## Backup no K8s
Podemos fazer backup das configuracoes dos agentes que estao instalados no cluster. Essas informacoes sao guardadas no ETCD Cluster que pode estar anexado a um Persistent Volume (PV).  
Como praticamente todos os agentes do K8s podem ser escritos atraves de arquivos YAML (forma declarativa) eh recomandado que esses arquivos YAML sejam versionados e salvos em repositorios. Com isso se voce perder tudo o que ha no seu cluster, voce tera como recriar aplicando facilmente.  
  
_Mas se alguem criar um agente de forma imperativa sem passar por uma documentacao?_  
Podemos fazer uma busca pela `kube-api-server` aplicando da seguinte forma:  
```sh
kubectl get pod -n meu-namespace -o yaml > meu-pod.yaml
```  
Dessa maneira podemos obter o arquivo de configuracao do Pod que foi criado sem documentar.  
  
## Ferramentas para backup dos agentes
Existem ferramentas que podem nos ajudar com o processo de backup desses agentes:  
- [Velero](https://velero.io/)  
  
## Backup no ETCD
O cluster ETCD eh instalado no Master Node. Quando instalamos o Kubernetes definimos o diretorio de onde o ETCD vai salvar os estados dos agentes do cluster. Podemos fazer backup desses arquivos tambem!  
Por padrao o banco ETCD ja vem com uma CLI para nos ajudar nisso.  
Habilite o `etcdctl` como cliente:  
```sh
export ETCDCTL_API=3
```  
Se executarmos:  
```sh
etcdctl snapshot save meu-backup.db
```  
Com isso o ETCD ira criar um arquivo chamado `meu-backup.db` com o backup do banco.  
Eh possivel ver o status do backup atraves do comando:  
```sh
etcdctl snapshot status
```  
  
### Como resgatar e aplicar os dados do backup feito do ETCD
Os seguintes passos devem ser feitos:  
```sh
service kube-apiserver stop
etcdctl snapshot restore meu-backup.db --data-dir /meu/path/do/backup
```  
Depois disso edite o arquivo de configuracoes do service ETCD para que use o diretorio do arquivo de backup. Edite a flag `--data-dir=/meu/path/do/backup`.  
Apos isso reinicie o ETCD.  
```sh
systemctl daemon-reload
service etcd restart
```  
Por ultimo, reinicie o `kube-api`:  
```sh
service kube-apiserver start
```  
  
*EXEMPLO*
```sh
etcdctl snapshot restore /opt/meu-backup.db --endpoints=127.0.0.1:2379 --cacert=/etc/kubernetes/pki/etcd/ca.crt --cert=/etc/kubernetes/pki/etcd/server.crt --key=/etc/kubernetes/pki/etcd/server.key --data-dir=/var/lib/etcd-backup
```  
Agora devemos editar o volume no arquivo `/etc/kubernetes/manifests/etcd.yaml`:
```vim
  volumes:
  - hostPath:
      path: /var/lib/etcd-backup
      type: DirectoryOrCreate
    name: etcd-data
```  
Restarte o pod do ETCD:  
```sh
kubectl delete pod etcd-controlplane -n kube-system
```  
_Vale lembrar que o ETCD eh um Pod estatico, por isso o arquivo de configuracao dele fica no diretorio /etc/kubernetes/manifests._  

## IMPORTANTE
Caso esteja usando certificado no seu cluster ETCD nao esqueca de passar como parametro os arquivos de CA, cretificado e chave privada ao executar comando `etcdctl`.  
  
## DICA DE CKA
Durante o exame voce tera que verificar se voce criou o agente que foi pedido. Logo, por exemplo, se foi pedido para que voce crie um Pod, voce tera que verificar atraves do comando `kubectl describe` se o agente que voce criou esta de acordo com o que foi pedido.
