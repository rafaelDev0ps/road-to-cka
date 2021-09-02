# Formas de autenticacao no cluster
Existem quatro principais que serao discutidas agora.

### Node
Quando o kubelet faz chamadas para o kube-api server antes de chegar na API essas chamdas passam por um modulo chamado Node Authorizer. Como vimos na parte de certificados, o kubelet utiliza certificados com grupos `system:nodes`, com isso as chamadas que possuem certificados com esse grupo passam pelo Node Authorizer.  
  
### ABAC (Atribute Based Access Conditions)
Nesse metodo de autenticacao eh vinculado um usuario com um conjunto de permissoes. Porem, eh o metodo mais trabalhoso pois cada usuario novo a ser adicionado eh necessario fazer o restart do kube-api server.  
  
### RBAC (Role Based Access Conditions)
Nesse caso podemos atribuir uma Role para um usuario ou grupo de usuarios do cluster. Bem mais organizado e mais simples!  
  
## Webhook
Usando o Webhook o kube-api server faz uma chamada para um agente externo responsavel pela permissao de ca usuario. Esse agente retorna uma resposta com a informacao de que esse user pode ou nao realizar determinada operacao.  
  
Vale lembrar que existem mais duas formas de autenticacao no cluster, sao elas: `AlwaysAllow` e `AlwaysDeny` como o proprio nome ja diz, um metodo permite que todos possam fazer qualquer operacao no cluster e outra restringe tudo.  
  
## Onde eh configurado a forma de autenticacao do cluster
Podemos configurar essas formas no Pod estatico do kube-api server.  
```yaml
  - command:
    - kube-apiserver
    - --advertise-address=192.168.1.31
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC  # <=== AQUI
    - --client-ca-file=/etc/kubernetes/pki/ca.crt
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key
    - --kubelet-preferred-address-types=InternalIP,ExternalIP,Hostname
    - --proxy-client-cert-file=/etc/kubernetes/pki/front-proxy-client.crt
    - --proxy-client-key-file=/etc/kubernetes/pki/front-proxy-client.key
    - --requestheader-allowed-names=front-proxy-client
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --requestheader-extra-headers-prefix=X-Remote-Extra-
    - --requestheader-group-headers=X-Remote-Group
    - --requestheader-username-headers=X-Remote-User
    - --secure-port=6443
    - --service-account-issuer=https://kubernetes.default.svc.cluster.local
    - --service-account-key-file=/etc/kubernetes/pki/sa.pub
    - --service-account-signing-key-file=/etc/kubernetes/pki/sa.key
    - --service-cluster-ip-range=10.96.0.0/12
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key
```  
No exemplo acima o cluster esta configurado para usar os metodos Node e RBAC de autorizacao.  
  
## Importante
Sempre que houver mais de um metodo de autorizacao configurado o kube-api server ira consultar um metodo por vez. Por exemplo, se atraves do Node Authorizer a autorizacao nao for permida o kube-api ira tentar de outra forma usando o RBAC. E assim por diante com os demais metodos configurados.  
