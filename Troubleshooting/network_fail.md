# Problemas na Rede  
Quando acontece falha na Rede do cluster o que devemos fazer para debuggar?  
  
O Kubernetes usa plugins CNI para configurar a rede. O kubelet é responsável por executar plugins, pois mencionamos os seguintes parâmetros na configuração do kubelet.  
  
- `cni-bin-dir`: o Kubelet investiga este diretório em busca de plugins na inicialização
- `network-plugin`: O plugin de rede a ser usado a partir de cni-bin-dir. Ele deve corresponder ao nome relatado por um plugin detectado no diretório de plug-ins.  
  
No exame CKA e CKAD, você não será solicitado a instalar o plugin CNI. Mas, se solicitado, você receberá o url exato para instalá-lo. Se não, você pode instalar a `weave-net` da documentação.  
  
