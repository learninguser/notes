# Different storage types on Kubernetes

## Storage on K8s

- There are multiple ways

1. emptyDir - ephemeral, as it is inside pod
2. hostPath - ephemeral, as it is inside server
3. Static provisioning
4. Dynamic provisioning

- Static and Dynamic provisioning are external storages, therefore the data on it is permanent

### 1. emptyDir

- As long as the pod is alive, we can use sidecar to read the logs and push them to Elasticsearch for further analysis
- If the pod is deleted, the logs are also deleted with this approach
- In case if they're pushed to Elasticsearch, the logs are still persistent
- emptyDir is the default K8s ephemeral volume type that is widely used in sidecar patterns which is mounted in both main and sidecar container
- emptyDir and sidecar patterns is for shipping the pod logs
- Filebeat is popular log shipping model from K8s to Elasticsearch
- Therefore it needs some configuration information:
  1. Where to ship ? elastic search server IP address
  2. What are the files to ship?
- We can provide this configuration through configmap
- [Source Code](https://github.com/sivadevopsdaws74s/k8-resources/blob/master/storage/01-empty-dir.yaml)

### 2. hostPath

- Using emptyDir, we can only ship the logs of the pod to the external system
- Sometimes we have to ship the logs of the servers for e.g. all the worker nodes for further analysis
- We use hostPath for shipping the server logs
- As we wanted to analyse the logs of more than one worker node, we use DaemonSet

#### DaemonSet

- If we run a DaemonSet, kubernetes will create a pod in each and every worker node.
- hostPath access is only given to administrators
- The purpose is to ship the server logs to Elasticserach even though it is as potential risks involved
- The only purpose of the pods created by Daemonset is to access te server logs
- It has a component called **fluentd** which acts as a shipper to access and send log files to Elasticsearch through hostPath
- As it can access all the files on the server, we can restrict it
- Name of the Admin Namespace: **kube-system**
- [Source Code](https://github.com/sivadevopsdaws74s/k8-resources/blob/master/storage/02-hostpath.yaml)

### 3. Static provisioning

- Storing logs in an external storage such as EBS
- As a part of static provisioning, create an EBS volume and give access to the EKS cluster
- From now on, EKS pods will store data in EBS volumes so that even though pod is deleted, the logs data are still there
- We use Persistent volume (PV) which is a representation of the underlying storage
- To use this, we create an equivalent PV object to represent EBS disk and map this to a EBS volume
- To access the EBS volume inside K8s, we need to install [EBS CSI drivers](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md#deploy-driver)
- To use the EBS storage inside K8s pods, we need to have **PersistentVolumeClaim**
- In Static provisioning, the storageClassName should be set to ""
- Once the pods that are writing to EBS volume are deleted, the EBS volume will be unmounted as well i.e. changed from In-use to available state on AWS
- [Source Code](https://github.com/sivadevopsdaws74s/k8-resources/blob/master/storage/03-ebs-static.yaml)

#### Steps involved

1. Create an EBS volume
2. Install EBS CSI drivers
3. EBS PV
4. Add necessary permissions to the worker nodes to access EBS
5. create PVC --> a way of claiming storage
6. create volume out of PVC
7. mount to container

- All the steps involved in static provisioning are manual whereas in dynamic provisioning, the disk is created automatically and mounted too using the storageClassName parameter
- **The data from the databases should be stored in the external storage but not on the server**

## Elastic File Storage (EFS)

- It is an implementation of NFS (Network File Storage)
