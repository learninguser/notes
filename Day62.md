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
2. Allow traffic in the security group of the EFS
    - Install necessary EFS drivers
3. Add AmazonElasticFileSystemFullAccess policy to any of the Worker Node to access EFS
4. Create PV, PVC and mount to pod

### Dynamic Provisioning using EFS

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

- Every server has its own database
- Let's consider a use-case where a user creation, updation or deletion request is received by the Master node, it assigns that task to any of the work nodes in the cluster
- The node that gets assigned will also inform all the other nodes about this request so that the data is persistent across all nodes in the cluster
- Stateful applications should have same name for all the worker nodes whereas with Deploymentset the nodes get assigned with a random name
- Statefulset will create pods in orderly manner i.e. nginx-0, nginx-1, nginx-2 will be created in an order.
  - If nginx-0 is successful only then it creates nginx-1 similarly nginx-2
- Deploymentset can create many pods at a time
- Statefulset keeps the pod identity same for the communication.
- Statefulset preserves network identity such as pod names
- Deployment will use service, but for statefulset it is mandatory to use headless service
- deployment and attach it to service
  - in deployment nslookup nginx-service will give IP address of service
- in statefulset nslookup nginx-service returns any of the Worker node IP address
