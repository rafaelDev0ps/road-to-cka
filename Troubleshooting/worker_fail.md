# Problemas no Worker Node
Quando acontece falha no Worker Node o que devemos fazer para debuggar?  
  
_Antes de tudo, entre dentro do Node via SSH_
  
- Verificar o status condition do Node (`describe`)
- Verifique os processos sendo executados no Node (comando `top`)  
- Verifique o armazenamento do Node (comando `df -h`) 
- Verifique o status e logs do `kubelet` (`journalctl -u kubelet -f`)
- Checar certificados TLS do Node  
- Validar arquivo de configuração em `/etc/kubernetes/kubelet.conf`
  
**IMPORTANTE**  
NÃO SE ESQUEÇA DE ESTAR DENTRO DO WORKER NODE ANTES DE FAZER O DEBUG!  
  
FIQUE ATENTO COM A PORTA DE CONEXÃO COM O MASTER NODE, A PORTA USADA É `6443`