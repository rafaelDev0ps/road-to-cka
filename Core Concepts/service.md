## Service
Services ajudam a aplicação conectarem com outros serviços externos e internos. Eles permitem que os pods tenham menos dependência com outros pods.  
  
Componentes importantes:  
- O node possui um IP
- O PC local possui outro IP
- A rede do pod tem um range de IP
- O pod tem um IP
  
O service é um serviço pequeno (um pod) que escuta em determinada porta e passa adiante a requisição feita para o serviço. Fazendo o famoso, port-forward.  
  
Tipos de Service
- NodePort: quando se acessa o Pod pelo IP do Node + Porta do Node
- ClusterIP: quando serviços precisam se comunicar internamente entre si. Cada Pod faz a chamada para o Service de outro Pod.
- LoadBalancer: quando precisa centralizar as chamadas feitas para vários IPs.
  
Terminologias
- Port: porta do Service
- TargetPort: porta do Pod
- NodePort: porta do Node (30000 - 32767)
  
### IMPORTANTE
Para vincular um pod com um service é necessário adicionar selectors!  

### OBSERVAÇÕES
O service consegue identificar mais de uma instancia de um mesmo Pod, fazendo um RoundRobin para alcança-los.  
Quando é iniciado o cluster Kubernetes um service do tipo ClusterIP é criado por padrão.  
