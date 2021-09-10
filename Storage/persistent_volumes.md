# Persistent Volumes (PV)
Os Persistent Volumes sao grandes espacos de armazenamento no cluster onde tem a finalidade de centralizar e compartilhar partes desse grande volume com as aplicacoes que estao rodando no cluster.  
  
Para que as aplicacoes consigam usar uma parte desse volume eh necessario que elas usem um PVC, um _Persistent Volume Claim_.  
  
Vejamos um exemplo, primeiro criamos um PV:
```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: meu-pv
spec:
  capacity:
    storage: 20Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/data
```  
  
E agora precisamos criar o Claim para que esse volume `meu-pv` possa ser usado no Node.  
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: meu-pvc
spec:
  accessModes:
    - ReadWriteOnce
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```  
  
Para anexar um PVC a um Pod fazemos:  
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
spec:
  containers:
  - name: nginx
    image: nginx:alpine
    volumeMounts:
      - name: meu-storage
        mountPath: /var/www/html
  volumes:
    - name: meu-storage
      persistentVolumeClaim:
        claimName: meu-pvc  # <== AQUI
```

**Dica!**  
Imagine que um PVC serve como um ticket para o PV disponibilizar armazenamento para o Node!  
  
Se por acaso voce delete o PVC, por padrao o Kubernetes ira manter os dados no PV ate que o admin remova manualmente os dados do PV. Podemos configurar esse comportamento atraves do campo `persistentVolumeReclaimPolicy` voce pode encontrar mais sobre isso [aqui](https://kubernetes.io/docs/tasks/administer-cluster/change-pv-reclaim-policy/).  
  
### Importante
Caso um PVC tente ser deletado, ele sera deletado somente se o Pod que estiver usando o PV for deletado tambem. Assim, o PV estara no estado `Retain` por padrao ou no padrao que foi configurado.