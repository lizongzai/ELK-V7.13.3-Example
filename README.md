# ELK-V7.13.3-Example
## Filebeat + ELK Stack Tutorial With Kubernetes



`ArtifactHub Official`

```bash
# Kibana Helm Chart
https://artifacthub.io/packages/helm/elastic/kibana

# Logstash Helm Chart
https://artifacthub.io/packages/helm/elastic/logstash

# Filebeat Helm Chart
https://artifacthub.io/packages/helm/elastic/filebeat

# Elasticsearch Helm Chart
https://artifacthub.io/packages/helm/elastic/elasticsearch
```



`Installing Helm (Ubuntu)`

```bash
https://helm.sh/docs/intro/install/
```

Members of the Helm community have contributed a [Helm package](https://helm.baltorepo.com/stable/debian/) for Apt. This package is generally up to date.

```bash
curl https://baltocdn.com/helm/signing.asc | gpg --dearmor | sudo tee /usr/share/keyrings/helm.gpg > /dev/null
sudo apt-get install apt-transport-https --yes
echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" | sudo tee /etc/apt/sources.list.d/helm-stable-debian.list
sudo apt-get update
sudo apt-get install helm
```



> - [ ] Create a customized directory for storing data based on project requirements, so here we create a new directory and give it permission, as the following path:
>
>   ```bash
>   $ mkdir /tmp/share/elasticsearch/data/nodes
>   $ chown -R 1000:1000 /usr/share/elasticsearch/data/nodes
>   ```
>
> - [ ] Before installing Elastic-search, you need to create an available PV and PVC and then bind them.
>
> - [ ] Finally, change the mountPath directory in the contents of the statefullset file to the previously created path for storing data.
>
>   ```yaml
>   volumeMounts:
>   - mountPath: "/tmp/share/elasticsearch/data/nodes"
>   ```

`elasticsearch-pv.yaml`

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-master-pv-0
  labels:
    app: elasticsearch-master
spec:
  storageClassName: k8s-kibana-logs
  capacity:
    storage: 30Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: default
    name: elasticsearch-master-elasticsearch-master-0
  hostPath:
    path: "/tmp/share/elasticsearch/data/nodes"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-node1.lab.example.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-master-pv-1
  labels:
    app: elasticsearch-master
spec:
  storageClassName: k8s-kibana-logs
  capacity:
    storage: 30Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: default
    name: elasticsearch-master-elasticsearch-master-1
  hostPath:
    path: "/tmp/share/elasticsearch/data/nodes"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-node2.lab.example.com
---
apiVersion: v1
kind: PersistentVolume
metadata:
  name: elastic-master-pv-2
  labels:
    app: elasticsearch-master
spec:
  storageClassName: k8s-kibana-logs
  capacity:
    storage: 30Gi
  accessModes:
    - ReadWriteOnce
  volumeMode: Filesystem
  persistentVolumeReclaimPolicy: Retain
  claimRef:
    namespace: default
    name: elasticsearch-master-elasticsearch-master-2
  hostPath:
    path: "/tmp/share/elasticsearch/data/nodes"
  nodeAffinity:
    required:
      nodeSelectorTerms:
      - matchExpressions:
        - key: kubernetes.io/hostname
          operator: In
          values:
          - k8s-node1.lab.example.com
```



`elasticsearch-pvc.yaml`

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: elasticsearch-master
  name: elasticsearch-master-elasticsearch-master-0
  namespace: default
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
  volumeMode: Filesystem
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 30Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: elasticsearch-master
  name: elasticsearch-master-elasticsearch-master-1
  namespace: default
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
  volumeMode: Filesystem
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 30Gi
---
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  labels:
    app: elasticsearch-master
  name: elasticsearch-master-elasticsearch-master-0
  namespace: default
spec:
  accessModes:
  - ReadWriteOnce
  resources:
    requests:
      storage: 30Gi
  volumeMode: Filesystem
status:
  accessModes:
  - ReadWriteOnce
  capacity:
    storage: 30Gi
```



`elasticsearch-master.yaml`

```yaml
kubectl edit statefulsets.apps elasticsearch-master
apiVersion: apps/v1
kind: StatefulSet
metadata:
  ...
        terminationMessagePath: /dev/termination-log
        terminationMessagePolicy: File
        volumeMounts:
        - mountPath: "/tmp/share/elasticsearch/data/nodes"
          name: elasticsearch-master
      dnsPolicy: ClusterFirst
    ...
```



```bash
$ helm install elasticsearch elasticsearch
$ helm install filebeat filebeat
$ helm install logstash logstash
$ helm install kibana kibana
```





> - [ ] Changes the LoadBalancer of the type of the Kibana content to `NodePort`
>
>   ```bash
>   $ kubectl edit svc kibana-kibana
>   ...
>   type: NodePort
>   ...
>   ```
>
> - [ ] Open your browser and enter Kibana URL: https://<node-ip>:<nodeport>, like the following:
>
>   ```
>   http://192.168.126.100:31533
>   ```
  


> - [ ] For Example:  Now we deploy an application server named Nginx to the Kubernetes Cluster via GitOps, and open a browser to access the application server UI. meanwhile we can obtain the log information about user access to NGINX on the Kibana interface.  
  
  
![image](https://user-images.githubusercontent.com/49580847/170850683-2d5587bf-d6ee-410f-b682-64b3ff7fdc29.png)
  

![image](https://user-images.githubusercontent.com/49580847/170850689-b4e964a7-c055-4c03-91fd-3d8c9295c4f9.png)
  

![image](https://user-images.githubusercontent.com/49580847/170850712-13cd8708-8015-4fb1-9842-82aa2588aab6.png)


