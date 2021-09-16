# Ingress
Ingresses servem para centralizar em uma unica URL varios Services instalados no cluster. Quando uma chamada eh realizada para um Ingress ele recebe a requisicao e redireciona para o endereco e porta do Service da aplicacao desejada. Esse redirecionamento eh baseado na rota da URL.  
  
Alem disso o Ingress tambem eh responsavel pela seguranca das requisicoes, ele gerencia os certificados de borda SSL de chamadas externas ao cluster.  
  
Podemos fazer uma analogia de que o Ingress eh uma camada Load Balancer que fica a frente dos Services e Deploys que pode ser configurado como qualquer outro agente no Kubernetes.  
  
O Ingress fica exposto para agentes externos ao cluster, semelhante ao Service do tipo `NodePort`. Tambem eh possivel integrar um Load Balancer do provedor cloud (caso esteja usando).  
  
**_Ok, mas como colocamos o Ingress no cluster?_**  
Existem varias alternativas de Ingress, as mais conhecidas sao:  
- [NGINX](https://www.nginx.com/)
- [HAPROXY](http://www.haproxy.org/)
- [traefik](https://doc.traefik.io/traefik/)  
- [Istio](https://istio.io/)
  
Assim que escolhemos umas dessas solucoes, devemos configurar SSL, endpoints dos services e etc.  
  
Existem dois principais agentes por tras do Ingress:  
- `Ingress Controller`: responsavel pelo Pods com as solucoes instaladas.
- `Ingress Resources`: responsavel pelas regras aplciadas no Ingress.  
  
### **IMPORTANTE**  
O cluster Kubernetes nao vem um Ingress instalado por padrao, portanto voce precisa criar um.  
  
**Ok, ja que nao vem instalado como eu instalo no meu cluster?**  
Vamos la...  
  
## Ingress Controller  
Para instalarmos o Ingress Controller precisamos os seguintes agentes.  
- Deployment  
```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  replicas: 1
  selector:
    matchLabels:
      app: nginx-ingress
  template:
    metadata:
      labels:
        app: nginx-ingress
    spec:
      serviceAccountName: nginx-ingress
      containers:
      - image: nginx/nginx-ingress:1.12.1
        imagePullPolicy: IfNotPresent
        name: nginx-ingress
        ports:
        - name: http
          containerPort: 80
        - name: https
          containerPort: 443
        env:
        - name: POD_NAMESPACE
          valueFrom:
            fieldRef:
              fieldPath: metadata.namespace
        - name: POD_NAME
          valueFrom:
            fieldRef:
              fieldPath: metadata.name
        args:
          - -nginx-configmaps=$(POD_NAMESPACE)/nginx-config   # <== NAMESPACE DO POD DESTINO AQUI
          - -default-server-tls-secret=$(POD_NAMESPACE)/default-server-secret  # <== AQUI TAMBEM
```  
  
- Service.  
```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  type: NodePort 
  ports:
  - port: 80
    targetPort: 80
    protocol: TCP
    name: http
  - port: 443
    targetPort: 443
    protocol: TCP
    name: https
  selector:
    app: nginx-ingress
```  
  
- Service Account.  
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: nginx-ingress 
  namespace: nginx-ingress
```  
O Service Account serve para conseguir aplicar o RBAC necessario para que o cluster funcione no cluster.  O RBAC necessario encontramos [aqui](https://github.com/nginxinc/kubernetes-ingress/tree/master/deployments/rbac).  
  
- Configmap.  
```yaml
kind: ConfigMap
apiVersion: v1
metadata:
  name: nginx-config
  namespace: nginx-ingress
data:
```  
Para editar as configuracoes do Load Balancer basta editar o Configmap.  
  
Depois de aplicar os Ingress Controller precisamos definir os Ingress Resources.  
  
### Ingress Resources
Nesse momento definimos como o Ingress ira se comportar ao redirecionar as requisicoes para os Services.  
```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: meu-ingress
spec:
  rules:
  - http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: meu-service
            port:
              number: 80
```  
Acima temos a configuracao basica de um Ingress que ira redirecionar para um unico Service chamado `meu-service` na porta `80` sempre que a requisicao foir feita na rota `/`.  
  
Podemos encontrar mais formas de configuracao do Ingress [aqui](https://kubernetes.io/docs/concepts/services-networking/ingress/).  
  
Podemos visualizar os Ingresses instalados no cluster.  
```sh
$ kubectl get ingress meu-ingress
```