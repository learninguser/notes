# Different storage types on Kubernetes

## Dynamic Provisioning Storage on K8s

- Creation of disks can be handled by Kubernetes.
- StorageClass (SC) can be used for creating the underlying disks and PersistentVolumes (PV) automatically

### Steps for setup

1. Install drivers, because SC object use drivers to create volumes.
    - [EBS CSI drivers](https://github.com/kubernetes-sigs/aws-ebs-csi-driver/blob/master/docs/install.md#deploy-driver)
2. Claim the necessary volume using PersistentVolumeClaim
3. Deploy the pods and attach volume to it

## EFS

### EBS vs EFS

- EBS is blockstore, EFS is network storage
- EBS can't scale automatically, whereas EFS can get more space automatically
- As EFS is dynamic in nature, one whole company can have one EFS
- Each folder gets an individual access point that is specific to it

### Static Provisioning using EFS

1. Create EFS filesystem
2. Allow traffic to Port 2049 in the security group of the EFS from worker nodes
3. Install necessary EFS drivers: `kubectl kustomize \
    "github.com/kubernetes-sigs/aws-efs-csi-driver/deploy/kubernetes/overlays/stable/ecr/?ref=release-1.7" > private-ecr-driver.yaml`
4. Add AmazonElasticFileSystemFullAccess policy to any of the Worker Node to access EFS
5. Create PV, PVC and mount it to the pod

### Dynamic Provisioning using EFS

- We need to have StorageClass for Dynamic Provisioning

1. Create EFS filesystem
2. Allow traffic in the security group of the EFS
    - Install necessary EFS drivers
3. Add AmazonElasticFileSystemFullAccess policy to any of the Worker Node to access EFS

- There is a folder structure that is created by K8s inside the access point which is not visible

## Statefulset

- So far, we have discussed:
  - ReplicaSet
  - DeploymentSet --> stateless applications i.e. data is not at all important i.e. cart, catalogue
  - DeamonSet
- Statefulset: used for DB related applications such as MySQL, Redis, MongoDB, Rabbimq

### Deploymentset vs Statefulset

- Let's consider a use-case where a user creation, updation or deletion request is received by the Master node, it assigns that task to any of the work nodes in the cluster
- The node that gets assigned **will also inform all the other nodes** about this request so that the data is persistent across all nodes in the cluster
- Stateful applications should have the same static hostname for all the worker nodes whereas with Deploymentset the nodes get assigned with a random name
- Statefulset will create pods in orderly manner i.e. nginx-0, nginx-1, nginx-2 will be created in an order.
  - If nginx-0 is successfully up and running only then it creates nginx-1 similarly nginx-2
  - This is to ensure that the cluster is successfully created
- Stateful applications should follow orderly provisioning and terminating
  - Provisioning happens in forward direction and termination happens in reverse direction
- In StatefulSet each Replica should have its own database
- Deploymentset can create many pods at a time
- Statefulset keeps the pod identity same for the communication.
- Statefulset preserves network identity such as pod names
- For e.g. when deploying MySQL database in Kubernetes, MySQL takes care of the communication in background but we should provision the storage using PV and PVC

#### How does statefulSet updates all the other nodes about the request?

- It uses a **Headless service** for this purpose i.e. it performs `nslookup <service>` to fetches all the IP address of the other nodes that are part of it
- Then it updates one-by-one with this information
- Whereas with Deployment uses a service and when we perform `nslookup`, it returns the IP address of service that redirects the traffic to any-of-the node
- **In statefulset, clusterIP should be set to None**
- When the pods with type Statefulsets are deleted, the storage associated with them are not deleted
- This is to ensure that the volumes are attached back when they're created once again

#### Steps to implement

1. Install drivers
2. Attach **AmazonEBSCSIDriverPolicy** policy to the IAM user attached to worker nodes
3. Install storageClass

- It is not suggested to store end user data inside K8s as it becomes very difficult to manage for the DB team
- We can store data such as logs inside Elasticsearch, prometheus etc with in the K8s
