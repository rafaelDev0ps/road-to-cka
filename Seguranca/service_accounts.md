# Service Accounts
Existem dois tipos de contas no Kubernetes, as UserAccounts (para usuarios) e os ServiceAccounts (para maquinas).  
ServiceAccounts sao as permissoes que uma aplicacao pode ter para executar uma determinada acao dentro do cluster. Por exemplo, ferramentas de monitoramento como ELK ou entao ferramentas de CI/CD como Gitlab.  
  
Para criar um ServiceAccount:  
```sh
kubectl create serviceaccount meu-service-account
```  
Quando criamos um ServiceAccount o K8s cria automaticamente um token para ser usado pela aplicacao. Com esse token serve para a aplicacao usar para autenticar dentro do cluster. **O token eh guardado em formato de secret no cluster**.  
  
_Para ficar mais pratico eh possivel pegar essa Secret com o Token e montar como um volume dentro do Pod da aplicacao, dessa maneira voce automatiza a utilizacao do token._  
  
Por falar nisso, todo namespace possui um ServiceAccount chamado `default`. A secret desse ServiceAccount sempre sera montada automaticamente nos Pods que voce cria, sem precisar declarar no manifesto.  
Geralmente a secret eh montada no diretorio `/.var/run/secrets/kubernetes.io/serviceaccount/`.  
Esse token do ServiceAccount `default` eh bem restrito, serve para executar apenas algumas buscas na API do Kubernetes. Portanto, eh recomendado criar o proprio ServiceAccount para a aplicacao.  
  
Podemos tambem restringir o mount automatico da secret do ServiceAccount `default` colocando o atributo `automountServiceAccountToken: false` dentro do campo `spec` do Pod.