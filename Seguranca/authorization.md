# Autenticacao
Por padrao o Kubernetes nao gerencia usuarios nativamente. Ele utiliza arquivos relacionados ao usuarios como o LDAP.
  
Nesse caso ele utiliza `Service Accounts`, que sao arquivos responsaveis por gerenciar as acoes que determinada conta pode fazer dentro do cluster.  
Podemos listar os service accounts executando:  
```sh
kubectl get serviceaccounts
# ou
kubectl get sa
```  
  
Todo acesso dos usuario sao controlados pelo kube-apiserver. Existem 4 principais formas de autenticacao no cluster:  
- Arquivo de senha
- Arquivo de token
- Certificados
- Servicos de Identidade (IAM)  
  
### Autenticacao Basica
Para configurar uma autenticacao de usuario/senha devemos definir um arquivo CSV com essas informacoes passando pela flag `--basic-auth-file` das configuracoes do `kube-apiserver.service`.  
Voce pode editar esse Pod estatico no arquivo `/etc/kubernetes/manifests/kube-apiserver.yaml`.  

```yaml
kube-apiserver
--advertise-address=192.168.1.31
--allow-privileged=true
--authorization-mode=Node,RBAC
--client-ca-file=/etc/kubernetes/pki/ca.crt
--enable-admission-plugins=NodeRestriction
--enable-bootstrap-token-auth=true
--etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
--etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
--etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
--etcd-servers=https://127.0.0.1:2379
--insecure-port=0
--kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
--kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
--kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
--proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
--proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
--requestheader-allowed-names=front-proxy-client
--requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
--requestheader-extra-headers-prefix=X-Remote-Extra-
--requestheader-group-headers=X-Remote-Group
--requestheader-username-headers=X-Remote-User
--secure-port=6443
--service-account-issuer=https://kubernetes.default.svc.cluster.local
--service-account-key-file=/etc/kubernetes/pki/sa.pub
--service-account-signing-key-file=/etc/kubernetes/pki/sa.key
--service-cluster-ip-range=10.96.0.0/12
--tls-cert-file=/etc/kubernetes/pki/apiserver.crt
--tls-private-key-file=/etc/kubernetes/pki/apiserver.key
--basic-auth-file=MEU-ARQUIVO-COM-USUARIO-SENHA.csv
```  
Note a ultima linha do trecho acima passa a configuracao explicada anteriormente.  
O formato do arquivo deve ser da seguinte maneira:  
```csv
senha1,user1,username1
senha2,user2,username2
senha3,user3,username3
```  
  
Quando voce editar as configuracoes do kube-apiserver ele ira reiniciar com essas novas alteracoes.  
Alem disso e possivel passar a coluna de `grupo` que cada usuario faz parte.  
Ao inves de utilizar a senha em formato de texto puro, podemos passar um token estatico no lugar da coluna de senha, com isso para acessar o cluster sera necessario passar esse token na requisicao como `--header "Authorization: Bearer <token>"`.  
  
_Vale lembrar que nao e uma boa pratica utilizar esse metodo de autenticacao!_  
  
Apos configurar a autenticacao basica podemos configurar as permissoes dos users.  
Primeiro criamos um Role definindo as permissoes do user:  
```yaml
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: default
  name: permissao-para-ler-pods
rules:
- apiGroups: [""] # "" indica o grupo Core API
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
```  
Depois disso aplicamos essa regra para o usuario existente atraves de uma RoleBinding.  
```yaml
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  name: permissao-para-ler-pods
  namespace: default
subjects:
- kind: User
  name: user1 # Nome eh case sensitive
  apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: Role #Role ou ClusterRole
  name: permissao-para-ler-pods # aqui o nome Role ou ClusterRole devem ser os mesmos
  apiGroup: rbac.authorization.k8s.io
```
  
Para autenticar no cluster voce deve fazer o seguinte request:  
```sh
curl -v -k https://localhost:6443/api/v1/pods -u "user1:senha1"
```  
