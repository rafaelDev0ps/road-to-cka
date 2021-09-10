# Container Storage Interface
No Kubernetes voce pode escolhar qual CRI quer que seu cluster trabalhe.  
Uma CRI significa _Container Runtime Interface_. Voce pode optar por 3 no K8s:  
- Docker
- rkt
- cri-o  
  
Alem disso existe a mesma opcao para suporte de rede. Os CNI, _Container Network Interface_ podem ser escolhidos em 3 no K8s:  
- weaveworks
- flannel
- cilium  
  
E os CSI, _Container Storage Interface_ para gerenciar os volumes dos cluster, que sao:  
- portworx
- Amazon EBS
- Dell EMC
- GlusterFS  
  
Por padrao o Kubernetes utiliza o Apache Mesos como CSI.