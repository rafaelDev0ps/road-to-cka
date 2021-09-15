# Pod Networking
Seguindo os conceitos do Kubernetes, os Pods por padrao devem conseguir se comunicar com qualquer outro Pod dentro do cluster independente do Node em que esteja.  
  
Essa comunicacao eh feita atraves de um CNI (Container Network Interface) que eh responsavel por criar um namespace de rede, interface virtual e conexoes entre os Pods. Ele segue o mesmo conceito quando vimos os [Network Namespaces](https://github.com/rafaelDev0ps/road-to-cka/blob/main/Networking/basics.md#network-namespaces).  
  
## CNI
O CNI eh configurado atraves do kubelet que fica localizado no `/usr/bin/kubelet` ele le as configuracoes que estao localizadas no `/var/lib/kubelet/confing.yaml`.  
  
Nesse arquivo de configuracao existe um campo chamado `--cni` onde se configura o CNI desejado. Outro campo que precisa ser declarado eh o `--network-pluguin=cni` par indicar que o cluster fara o uso desse plugin para rede.  

Os arquivos binarios do CNI ficam `/opt/cni/bin` apos instalado.  
  
Quando configurado as informacoes sobre o CNI podem ser descobertas listando o processo no Node.  
```sh
$ ps -aux | grep kubelet
```  
Nele vera os paths para cada confuguracao mencionada anteriormente.  
  
### weave-net  
Podemos instalar um CNI chamado weave-net no nosso cluster.  
```sh
$ kubectl apply -f "https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version |
 base64 | tr -d '\n')"
```  
Dessa forma ele ira instalar os Pods e Daemonsets para gerenciarem o trafego dentro do cluster.  
  
Para saber mais sobre a instacacao do weave-net leia [aqui](https://www.weave.works/docs/net/latest/kubernetes/kube-addon/#-installation).  