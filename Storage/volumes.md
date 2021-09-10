# Volumes
No Kubernetes os dados sao descataveis, quando os Pods sao deletado os dados tambem sao removidos, alem disso, nao existe armazenamento fora do container caso nao tenha montado um volume no seu workload.  
  
Para persistir dados em um container eh necessario anexar um volume junto dele. Assim, se um container for removido, os dados que estao no volume irao persistir.  
  
Para anexarmos um volume a um Pod fazemos o que esta  no manifesto abaixo.  
```yaml 
apiVersion: v1
kind: Pod
metadata:
  name: meu-app
spec:
  restartPolicy: Never
  volumes:
  - name: meu-volume
    hostPath:
      path: /meus-dados
  containers:
  - name: meu-container
    image: "k8s.gcr.io/busybox"
    volumeMounts:
    - name: meu-volume
      mountPath: /teste
```  
Acima vemos um exemplo onde o Pod `meu-app` tem um volume anexado no container `meu-container` que tera um diretorio `/teste` com o conteudo que esta no `/meus-dados`.  
  
**Atencao!**  
Vale lembrar que se voce estiver usando volumes em um cluster eh recomendado que monte diretorios diferentes para cada aplicacao, pois os Pods podem rodar em mais de um Node e com isso os dados podem mudar de Node para Node. 