# Different storage types on Kubernetes

## Storage on K8s or K8s volumes

- The storage types can be broadly classified into 2 types:
  1. Internal
  2. External

1. emptyDir - ephemeral, as it is **inside** pod
2. hostPath - ephemeral, as it is **inside** server
3. Static provisioning
4. Dynamic provisioning

- Static and Dynamic provisioning are external storages, therefore the data on it is permanent

### 1. emptyDir

- As long as the pod is alive, we can use sidecar to read the logs and push them to Elasticsearch for further analysis
- If the pod is deleted, the logs are also deleted with this approach
- In case if they're pushed to Elasticsearch, the logs are still persistent
- emptyDir is the default K8s ephemeral volume type that is widely used in sidecar patterns and is mounted in both main and sidecar container
- emptyDir and sidecar patterns is for shipping the pod logs
- Filebeat is popular model to ship the logs from K8s to Elasticsearch
- Filebeat is a public image from Elasticsearch
- Therefore it needs some configuration information:
  1. Where to ship ? Elastic search server IP address
  2. What are the files to ship?
- We can provide this configuration through **ConfigMap**
- [Source Code](https://github.com/daws-76s/k8-resources/blob/main/storage/01-empty-dir.yaml)

### 2. hostPath

- Using emptyDir, we can only ship the logs of the pod to the external system
- Sometimes we have to ship the logs of the servers for e.g. all the worker nodes for further analysis
- For this purpose, we use hostPath for shipping the server logs
- As we wanted to analyse the logs of more than one worker node, we use **DaemonSet**

#### DaemonSet

- If we run a DaemonSet, kubernetes will create a pod in each and every worker node. Also it deletes those pods when the node is deleted
- hostPath access is only given to administrators
- The purpose is to ship the server logs to Elasticsearch even though it is has potential risks involved
- The only purpose of the pods created by Daemonset is to access the server logs
- For this purpose, we can use **fluentd** from ELK Stack which will be deployed as a Daemonset that can access the underlying host logs through hostPath and send them to Elasticsearch
- Fluentd continuosly monitors the `/var/log/` directory and ships them to Elasticsearch. But we can also modify it to access the complete filesystem which is dangerous. Therefore we should restrict it by only providing a read-only access
- **Name of the admin Namespace: kube-system**
- [Source Code](https://github.com/daws-76s/k8-resources/blob/main/storage/02-hostpath.yaml)

### 3. Static provisioning

- Storing logs in an external storage such as Elastic Block store (EBS) is safe and secure
- As a part of static provisioning, create an EBS volume and give access to the EKS cluster
- From now on, EKS pods will store data in EBS volumes so that even though pod is deleted, the logs data are still there
- Similarly we also have EFS (Elastic File System) on AWS
- The harddisks i.e. EBS should be created in the same AZ's as the location of servers
- To access the EBS volume inside K8s, we need to install [EBS CSI drivers](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md#deploy-driver)
- A proper role should be attached to EC2 instance in order to access the EBS

#### K8s volumes and resources

1. Persistent Volume (PV)
2. Persistent Volume Claim (PVC)

- We use **Persistent volume (PV)** object which is a representation of the underlying external storage i.e. a wrapper for the external storage in K8s as the administrators need not to understand the underlying architecture of the EBS Harddisk. Therefore K8s came up with this PV object which we can perform operations on the underlying storage
- To use this, we create an equivalent PV object to represent EBS disk and map this to a EBS volume
- To use the EBS storage inside K8s pods, we need to have **PersistentVolumeClaim (PVC)** i.e. Pods should request volume through PVC
- PVC is not a physical representation rather its just a request to the PV
- When creating a PV object, twe should set the access mode to **ReadWriteOnce** for the EBS volume
- Using PVC, the pod can claim a certain amount of space from the PV.
- In Static provisioning, the storageClassName should be set to `""` as we're creating it manually
- After claiming, we should attach it to the Pod.
- To see if its working fine or not, we can attach a loadBalancer service to the pod
- When a pod gets deployed on to the nodes, there is chance that it might get deployed in the node in another AZ in which the created EBS volume is not present
- To restrict this, we can use: **nodeSelector** and assign labels to the nodes and specify in which nodes the pod can be deployed based on the labels
- We can see all the current labels of the nodes using: `kubectl get nodes --show-labels`
- To label a node, `kubectl label nodes <NAME-OF-THE-NODE> <LABEL>`
- Once the pods that are writing to EBS volume are deleted, the EBS volume will be unmounted as well i.e. changed from In-use to available state on AWS
- [Source Code](https://github.com/sivadevopsdaws74s/k8-resources/blob/master/storage/03-ebs-static.yaml)

#### Steps involved

1. Create an EBS volume: Either storage admin or K8s admin will create this
2. Install EBS CSI drivers
3. EBS PV
4. Add necessary permissions to the worker nodes to access EBS
5. create PVC --> a way of claiming storage
6. create volume out of PVC
7. mount to container

### 4. Dynamic Provisioning

- Volume should be created automatically
- There is another object called **storageClass** that can create storage dynamically based on the request
- If we have created volume manually, we should also create PV and PVC objects manually and mount it in the pod
- As storageClass creates the external volume, it is its responsiblity of creating the PV object as well
- We can get the list of storageClasses using: `kubectl get sc`
- All the steps involved in static provisioning are manual whereas in dynamic provisioning, the disk is created automatically and mounted too using the storageClassName parameter
- Since storageClass is a ClusterLevel resource, admins should create this and hence EBS will have one storageClass cluster wide.
- Once the storageClass is created by admins, they will inform other teams and they will clam the space as per the need.
- PV is not a namespaced resource whereas PVC is a namespaced one.
- To get all the PV and PVC information, we can use: `kubectl get pv`, `kubectl get pvc`
- If the pod is deleted, the underlying volume will also be deleted as the RECLAIMPOLICY is set to delete which can be observed from: `kubectl get sc` command output
- We can set to `Retain` when defining the StorageClass object i.e. `reclaimPolicy: Retain`
- **The data from the databases should be stored in the external storage but not on the server**

## Elastic File Storage (EFS)

- It is an implementation of NFS (Network File Storage)
- NFS works on TCP Port 2049
