# Storage Classes
Existem alguns tipos de storage. Vejamos...  
  
## Static Provisioning
Quando trabalhamos com PV e PVC precisamos antes de tudo provisionar o armazenamento para os Nodes do cluster. Se voce estiver usando algum provedor cloud para provisionar o armazenamento voce pode configurar isso no seu PV, seja com GCP, AWS e Azure por exemplo.  

## Dynamic Provisioning
Nesse caso quando se cria um PV o provedor automaticamente provisiona um armazenamento para seus Nodes. Para isso eh necessario criar um `StorageClass`.  
```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: meu-storage-aws
provisioner: kubernetes.io/aws-ebs
```  
Voce pode encontrar a lista de provedores que possuem integracao com Kubernetes [aqui](https://kubernetes.io/docs/concepts/storage/storage-classes/#provisioner).  
  
Depois disso precisamos definir esse StorageClass no nosso PVC.  
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: meu-pvc
spec:
  accessModes:
    - ReadWriteOnce
  storageClassName: meu-storage-aws  # <== AQUI
  volumeMode: Block
  resources:
    requests:
      storage: 10Gi
```  
  
### **Importante**
Existe um comportamento que se configura nos Storage Classes chamado `VolumeBindingMode` onde define a maneira como o volume sera provisionado. Por padrao, o valor desse campo eh definido como `WaitFormFirstConsumer`, ou seja, sera provisionado somente quando um Pod utilizar esse volume, caso contrario o PVC ficara em estado `Pending`.