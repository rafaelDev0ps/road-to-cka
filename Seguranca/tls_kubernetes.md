# Como funciona TLS dentro do Kubernetes?
Existem dois tipos de certificados no cluster. Os certificados dos clientes e os certificados dos servidores.
| Clientes | Servidores |
| --- | ---| 
| admin | ETCD Server |
| scheduler | kube-api server | 
| controller-manager | kubelet |
| kube-proxy |  |
| kubelet-client |  | 
| kube-api server client |  |
| api-server ETCD client |  |
  
## Importante
**O K8s recomenda que voce tenha mais de uma CA para assinar esses certificados elencados na tabela acima. Geralmente, encontramos dois certificados, um para os clientes e servidores e outro exclusivo para o ETCD server.**  
  
**Importante lembrar que somente o kube-api server se comunica com o ETCD cluster. Os demais modulos se comunicam com o kube-api server.**  
  
## Como gerar certificados para o cluster K8s?
Podemos usar a ferramenta `openssl` para gerar certificados.  
Primeiro criamos uma chave privada:
```sh
openssl genrsa -out ca.key 2048
```  
  
Depois, usamos a  chave privada para gerar a request para o certificado:  
```sh
openssl req -new -key ca.key -subj "/CN=KUBERNETES-CA" -out ca.csr
```  
_Um request para certificado eh como um certificado porem sem assinatura._  
  
Agora, vamos assinar nossso certificado:  
```sh
openssl x509 -req -in ca.csr -signkey ca.key -out ca.crt
```  
Com isso temos nossa CA e nossa chave privada!  
  
---
  
Agora vamos realizar o mesmo processo para o cliente `Admin`:
```sh 
openssl genrsa -out admin.key 2048
```  
  
Criamos o request para o certificado com um Common Name (CN) diferente **e atribuimos um grupo (O) para esse request para que ele seja identificado como um user admin no cluster**:  
```sh
openssl req -new -key admin.key -subj "/CN=kube-admin/O=system:masters" -out admin.csr
```  
  
**AGORA VEM O PULO DO GATO!**  
  
Vamos assinar esse request com a CA que ja foi criada para o Server:  
```sh
openssl x509 -req -in admin.csr -CA ca.crt -CAkey ca.key -out admin.crt
```  
Dessa maneira o admin ira conseguir se autenticar no cluster!  
  
### Importante 
O processo se repete para os demais clientes assim como feito no exemplo para o Admin.  
Tanto para o Cliente quanto para o Server eh necessario que eles cada um deles tenham uma copia da CA.  
  
---
  
Agora vamos gerar a chave e certificado para os `Servers`, vamos pegar o exmeplo do `kube-api server`. Ele eh o componente mais usado dentro do cluster, e possui varios nomes diferentes dentro cluster como `kubernetes`, `kubernetes.default.svc`, `kubernetes.default.svc.cluster.local`, `10.96.0.1` portanto precisamos colocar esses nomes no certificado para que as chamadas possam ser validadas corretamente.  
Portanto criamos a chave privada para ele:  
```sh
openssl genrsa -out apiserver.key 2048
```
  
Depois, geramos o request para o certificado:  
```sh
openssl req -new -key apiserver.key -subj "/CN=kube-apiserver" -out apiserver.csr -config openssl.cnf
```  
Para incluir todos os nomes alternativos dado ao `kube-api server` precisamos criar um arquivo do openssl chamado `openssl.cnf`:  
```vim
subjectAltName = @alt_names
[alt_names]
DNS.1 = kubernetes
DNS.2 = kubernetes.default
DNS.3 = kubernetes.default.svc
DNS.4 = kubernetes.default.svc.cluster.local
IP.1 = 10.96.0.1
IP.2 = 172.17.0.87
```  
Agora, basta assinar o certificado com a mesma CA criada:  
```sh
openssl x509 -req -in apiserver.csr -CA ca.crt -CAkey ca.key -out apiserver.crt
```

Agora precisamos colocar os paths dos certificados e chaves nas configuracoes do Pod estatico do `kube-api-server`
```yaml
  - command:
    - kube-apiserver
    - --advertise-address=192.168.1.31
    - --allow-privileged=true
    - --authorization-mode=Node,RBAC
    - --client-ca-file=/etc/kubernetes/pki/ca.crt   # <=== AQUI
    - --enable-admission-plugins=NodeRestriction
    - --enable-bootstrap-token-auth=true
    - --etcd-cafile=/etc/kubernetes/pki/etcd/ca.crt  # <=== AQUI
    - --etcd-certfile=/etc/kubernetes/pki/apiserver-etcd-client.crt  # <=== AQUI
    - --etcd-keyfile=/etc/kubernetes/pki/apiserver-etcd-client.key  # <=== AQUI
    - --etcd-servers=https://127.0.0.1:2379
    - --insecure-port=0
    - --kubelet-client-certificate=/etc/kubernetes/pki/apiserver-kubelet-client.crt  # <=== AQUI
    - --kubelet-client-key=/etc/kubernetes/pki/apiserver-kubelet-client.key  # <=== AQUI
    - --kubelet-certificate-authority=/etc/kubernetes/pki/ca.pem  # <=== AQUI
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
    - --tls-cert-file=/etc/kubernetes/pki/apiserver.crt  # <=== AQUI
    - --tls-private-key-file=/etc/kubernetes/pki/apiserver.key  # <=== AQUI
```

### Importante
**Nao se esqueca de passar os paths desses certificados na configuracoes do kube-api-server para que ele possa validar os certificados!**   
  
## Sobre os Worker Nodes
Sao tratados da mesma forma como os clientes demonstrados anteriomente. Quando for criar o request par o Certificado eh necessario atribuir o grupo `system:nodes` (e.g. `system:nodes:worker02`).  
  
## Onde consigo visualizar as configuracoes dos certificados?
Voce pode visualizar no arquivo:  
```sh
cat /etc/systemd.system/kube-apiserver.service
```  
Ou ent'ao visualizar as configuracoes do Pod estatico:  
```sh
cat /etc/kubernetes/manifests/kube-apiserver.yaml
```    
  
Depois que voce conseguiu coletar o path do certificado podemos usar o `openssl` para extrair as informacoes dele:  
```sh
openssl x509 -in /etc/kubernetes/pki/apiserver.crt -text -noout
```
  
Voce tambem pode inspecionar os logs dos services do cluster para saber mais sobre os certificados:  
```sh
journalctl -u etcd.service -l
# ou usando o kubectl
kubectl logs etcd-master
```  
  
## API de Certificados
O Kubernetes possui a propria API para gerenciar os certificados. Quando for necessario adicionar um novo usuario para conseguir acessar o cluster, precisamos enviar um request para que a API de Certificados do Kubernetes assine e o novo usuario consiga autenticar.  
Para isso vamos supor um cenario onde um novo usuario precisa acessar o cluster.  
1. O novo usuario cria uma chave privada para ele
2. O novo usuario criar um request para assinar o certificado dele
Junto disso o novo user precisa criar um manifesto YAML para assinar o certificado.  
```yaml
apiVersion: certificates.k8s.io/v1
kind: CertificateSigningRequest
metadata:
  name: novo-user
spec:
  request: LS0tLS1CRUdJTiBDRVJUSUZJQ0FURSBSRVFVRVNULS0tLS0KTUlJQ1ZqQ0NBVDRDQVFBd0VURVBNQTBHQTFVRUF3d0dZVzVuWld4aE1JSUJJakFOQmdrcWhraUc5dzBCQVFFRgpBQU9DQVE4QU1JSUJDZ0tDQVFFQTByczhJTHRHdTYxakx2dHhWTTJSVlRWMDNHWlJTWWw0dWluVWo4RElaWjBOCnR2MUZtRVFSd3VoaUZsOFEzcWl0Qm0wMUFSMkNJVXBGd2ZzSjZ4MXF3ckJzVkhZbGlBNVhwRVpZM3ExcGswSDQKM3Z3aGJlK1o2MVNrVHF5SVBYUUwrTWM5T1Nsbm0xb0R2N0NtSkZNMUlMRVI3QTVGZnZKOEdFRjJ6dHBoaUlFMwpub1dtdHNZb3JuT2wzc2lHQ2ZGZzR4Zmd4eW8ybmlneFNVekl1bXNnVm9PM2ttT0x1RVF6cXpkakJ3TFJXbWlECklmMXBMWnoyalVnald4UkhCM1gyWnVVV1d1T09PZnpXM01LaE8ybHEvZi9DdS8wYk83c0x0MCt3U2ZMSU91TFcKcW90blZtRmxMMytqTy82WDNDKzBERHk5aUtwbXJjVDBnWGZLemE1dHJRSURBUUFCb0FBd0RRWUpLb1pJaHZjTgpBUUVMQlFBRGdnRUJBR05WdmVIOGR4ZzNvK21VeVRkbmFjVmQ1N24zSkExdnZEU1JWREkyQTZ1eXN3ZFp1L1BVCkkwZXpZWFV0RVNnSk1IRmQycVVNMjNuNVJsSXJ3R0xuUXFISUh5VStWWHhsdnZsRnpNOVpEWllSTmU3QlJvYXgKQVlEdUI5STZXT3FYbkFvczFqRmxNUG5NbFpqdU5kSGxpT1BjTU1oNndLaTZzZFhpVStHYTJ2RUVLY01jSVUyRgpvU2djUWdMYTk0aEpacGk3ZnNMdm1OQUxoT045UHdNMGM1dVJVejV4T0dGMUtCbWRSeEgvbUNOS2JKYjFRQm1HCkkwYitEUEdaTktXTU0xMzhIQXdoV0tkNjVoVHdYOWl4V3ZHMkh4TG1WQzg0L1BHT0tWQW9FNkpsYWFHdTlQVmkKdjlOSjVaZlZrcXdCd0hKbzZXdk9xVlA3SVFjZmg3d0drWm89Ci0tLS0tRU5EIENFUlRJRklDQVRFIFJFUVVFU1QtLS0tLQo=
  usages:
  - server auth
  - key encipherment
  - digital signature
  groups:
  - system:authenticated
```  
**Note que no campo `request` eh preenchido com o resquest para assinar o certificado em formato `base64`.**  
  
3. Agora com o usuario admin buscamos e aprovamos o request do novo usuario:
```sh
kubectl get csr
kubectl certificate approve novo-usuario
```  
4. O usuario admin deve extrair o certificado do manifesto apos aprovar o request:  
```sh
kubectl get csr novo-usuario -o yaml -o jsonpath='{.certificate}' | base64 -d
```  
  
_Quem eh responsavel por assinar e validar os certificados que passam no cluster?_  
O `kube-controller-manager` eh o modulo responsavel por gerenciar os certificados. Podemos ver as configuracoes dele visualizando o Pod estatico:  
```sh
cat /etc/kubernetes/manifests/kube-controller-manager.yaml
```  
```yaml
  containers:
  - command:
    - kube-controller-manager
    - --authentication-kubeconfig=/etc/kubernetes/controller-manager.conf  # <=== AQUI
    - --authorization-kubeconfig=/etc/kubernetes/controller-manager.conf  # <=== AQUI
    - --bind-address=127.0.0.1
    - --client-ca-file=/etc/kubernetes/pki/ca.crt # <=== AQUI
    - --cluster-name=kubernetes
    - --cluster-signing-cert-file=/etc/kubernetes/pki/ca.crt # <=== AQUI
    - --cluster-signing-key-file=/etc/kubernetes/pki/ca.key # <=== AQUI
    - --controllers=*,bootstrapsigner,tokencleaner
    - --kubeconfig=/etc/kubernetes/controller-manager.conf
    - --leader-elect=true
    - --port=0
    - --requestheader-client-ca-file=/etc/kubernetes/pki/front-proxy-ca.crt
    - --root-ca-file=/etc/kubernetes/pki/ca.crt # <=== AQUI
    - --service-account-private-key-file=/etc/kubernetes/pki/sa.key
    - --use-service-account-credentials=true
```