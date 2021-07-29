## Basico de certificados
TLS garante que a comunicacao entre cliente e servidor sera segura.  
  
Quando nao usado pode ser alvo facil para snifing de hackers.  
  
Quando usamos certificados os dados sao encriptados por uma chave que soh pode ser usada
pelo server. Existe uma chave publica e uma chave privada, onde a public encripta e a 
privada decripta.  
  
Eh muito importante saber quem assina o certificado. Existem varias empresas confiaveis
geram certificados.  
  
Geralmente a nomenclatura para chave publicas tem a extensao .crt ou .pem enquanto as chaves privadas tem extensao .key ou -key.pem.  
  
## Certificados no Kubernetes
Certificados para os Clientes
- User Admin
- Scheduler
- Controller Manager
- Kube-proxy
- Kube-api client
- ETCD client
- Kubelet client

---

Certificados para os Servidores
- ETCD server
- Kube-API server
- Kubelet Server  
  
*OBS: para todos esses deve haver uma chave publica e uma privada*  

## Criacao de certificados
Passo 1: gerar as chaves  
Passo 2: fazer uma requisicao para assinar o certificado  
Passo 3: assinar o certificado  
