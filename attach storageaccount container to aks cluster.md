```yaml
---
apiVersion: v1
data:
  azurestorageaccountkey: <stroageaccount#_key>    # echo stroageaccount_key | base64
  azurestorageaccountname: <storageaccount name in base64> # echo storageaccountname | base64
kind: Secret
metadata:
  name: azure-secret
  namespace: <namespace>
type: Opaque
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: pv-az-test-1
spec:
  capacity:
    storage: 10Gi
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain  # "Delete" is not supported in static provisioning
  mountOptions:
    - -o allow_other
    - --file-cache-timeout-in-seconds=120
  csi:
    driver: blob.csi.azure.com
    readOnly: false
    volumeHandle: unique-volumeid
    volumeAttributes:
      resourceGroup: <ResourceGroup>
      storageAccount: <storageAccountName>
      containerName: <containerName>
    nodeStageSecretRef:
      name: azure-secret
      namespace: <namespace>
---
kind: PersistentVolumeClaim
apiVersion: v1
metadata:
  name: pvc-az-test-1
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 10Gi
  volumeName: pv-az-test-1
  storageClassName: ""
---
apiVersion: v1
kind: Pod
metadata:
  name: az-test-1
  namespace: <namespace>
spec:
  containers:
  - image: mcr.microsoft.com/oss/nginx/nginx:1.15.5-alpine
    imagePullPolicy: IfNotPresent
    name: mypod
    resources:
      limits:
        cpu: 250m
        memory: 256Mi
      requests:
        cpu: 100m
        memory: 128Mi
    volumeMounts:
    - mountPath: /mnt/azure
      name: azure
  volumes:
  - name: azure
    persistentVolumeClaim:
      claimName: pvc-az-test-1
      
 ```
