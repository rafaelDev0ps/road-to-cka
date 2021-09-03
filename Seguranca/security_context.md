# Security Context
Quando executamos um container Docker temos a opcao de configurar que tipo de usuario pode acessar e quais permissoes ele tem dentro do container. Isso pode ser configurado no K8s tambem.  
  
Existem dois contextos de seguranca para container no K8s. 
- container level
- pod level
  
Se as configuracoes forem escritas no contexto do Pod e no container, valera as configuracoes escritas no container.  
  
No contexto de Pod:  
```yaml
apiVersion: v1
kind: Pod
matadata: 
  name: meu-pod
spec:
  securityContext:
    runAsUser: 1000
  containers:
  - name: meu-app
    image: meu-app
```
  
No contexto de container:  
```yaml
apiVersion: v1
kind: Pod
matadata: 
  name: meu-pod
spec:
  containers:
  - name: meu-app
    image: meu-app
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
```  

Caso seja configurado nas duas maneiras:  
```yaml
apiVersion: v1
kind: Pod
matadata: 
  name: meu-pod
spec:
  securityContext:
    runAsUser: 1010
  containers:
  - name: meu-app
    image: meu-app
    securityContext:
      runAsUser: 1000
      capabilities:
        add: ["MAC_ADMIN"]
  - name: outro-app
    image: outro-app
```  
No caso acima o container `meu-app` sera executado com usuario `1000` enquanto o container `outro-app` sera executado como usuario `1010`.
  
### Importante
As `capabilities` soh funcionam no contexto de container!  
Por padrao as aplicacoes sao executadas como `root` user no docker.