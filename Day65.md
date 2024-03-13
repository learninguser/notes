# Further Topics in K8s

## Provision EKS cluster using Terraform

- EKS cluster is a VPC based resource
- Load Balancing is not a part of the EKS cluster
- **Each and every resource that is a part of VPC has a Security group associated with it**
- An IAM user is also created when creating the cluster with `eksctl` and we modified it according to our needs
- But when provisioning it using Terraform, we will automate it including the security groups that are linked with Node instances and Control plane
- All traffic should be enabled b/w control plane and worker node instances and vice-versa as it is completely secured and present inside a VPC
- The worker node instances should accept traffic on Ephemeral range from the loadbalancer as node port is created with in the range by EKS
- We can place the whole EKS clutser and worker nodes inside a Private subnet and Load balancer inside a public subnet
- **cluster_node**: Cluster accepting connections from Node
- After provisioning it using Terraform, we should update `kubeconfig` information in the workstation which can be done using: `aws eks update-kubeconfig --region us-east-1 --name roboshop-tf`

## Taints and Tolerations

### Taints

- Taint is nothing but paint i.e. polluting it
- When a node is tainted, the default scheduler can't schedule pod on that node
- But the exisiting pods in the node will continue running on it
- In organisation, few projects add their nodes to EKS cluster and would like to restrict other projects not to provision their pods inside their nodes.
- For this purpose, they taint their nodes
- We can do this using: `kubectl taint nodes <name of the node> key1=value1:NoSchedule`
  - `NoSchedule`: It specifies the scheduler not to schedule any new pods on that node
  - `NoExecute`: It deletes all the pods that are running on that node
  - `PreferNoSchedule`: K8s tries not to schedule new pods on the tained node but can't guarantee it
  - E.g. `kubectl taint nodes ip-10-0-11-33.ec2.internal project=roboshop:NoSchedule`
- To untaint a node, we can use: `kubectl taint nodes <name of the node> key1=value1:NoSchedule-`. For e.g. `kubectl taint nodes ip-10-0-11-33.ec2.internal project=roboshop:NoSchedule-`

### Tolerations

- A project can apply tolerations i.e. excuses inorder to schedule the pods in a specific node only
- We can specify the tolerations at the container level in the sepcifications of the resource

  `selectors/01-taint-toleration.yaml`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: taint-pod
  spec:
    # list of containers
    containers:
    - name: hello-pod
      image: nginx
      ports:
      - containerPort: 80
    tolerations:
    - key: "project"
      operator: "Exists"
      effect: "NoSchedule"
  ```

- Once a toleration is specified, the scheduler can provision the pods inside the tainted nodes that matches as well

## Affinity and anti-affinity

- Earlier when attaching an external EBS storage to a Pod, we used a PV object and we mentioned that both the EBS volume and nodes should be in the same region for the volume to be attached
- For this, we used `nodeSelector` attribute to specify the region and which nodes can be selected
- With `nodeSelector`, we don't have much options at hand to filter the nodes whereas with Affinity and anti-affinity, we have more options to select the nodes
- There are 2 types of node affinity:
  1. `requiredDuringSchedulingIgnoredDuringExecution`: While scheduling the pods, the labels should match and its a hardrule. But these settings are ignored not at the time of pods execution
  2. `preferredDuringSchedulingIgnoredDuringExecution`: While scheduling the pods, the scheduler tries to match the labels and if doesn't match, still the scheduler schedules the pod. These settings are ignored not at the time of pods execution
- Before applying affinity or anti-affinity, the nodes should be labeled. It can be done using: `kubectl label nodes ip-10-0-11-33.ec2.internal project=roboshop`

  `selectors/02-node-affinity.yaml`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: with-node-affinity
  spec:
    affinity:
      nodeAffinity:
        requiredDuringSchedulingIgnoredDuringExecution:
          nodeSelectorTerms:
          - matchExpressions:
            - key: project
              operator: In
              values:
              - roboshop
    containers:
    - name: with-node-affinity
      image: nginx
  ```

- Node affinity weight will only work for nodes with `preferredDuringSchedulingIgnoredDuringExecution` option
- Anti affinity is nothing but setting the value of operator to `NotIn`
- Similar to node affinity, we also have pod affinity. For e.g. lets say we have pod1 that is running in node1. Now when we set pod affinity to pod2, it would also like to run inside the same node where pod1 is running.

## Cluster Upgrade

- Before upgradation, you can announce downtime i.e. no Create, Update of applications, No releases or Changes
- Also modify the security group settings so that no one is able to access the cluster using `kubectl`

1. Create a seperate node group called green
2. We need to taint new worker nodes i.e. `kubectl taint nodes ip-10-0-11-157.ec2.internal project=roboshop:NoExecute` so that any running pods in those will be terminated
3. Upgrade control plane to 1.28 i.e. manually through console so that the existing blue worker node group will not be affected
4. Upgrade green node group to 1.28
5. Taint blue nodes so that new pods will not be scheduled
6. untaint green
7. cordon/drain the blue nodes using: `kubectl drain --ignore-daemonsets <node-name> --force --delete-emptydir-data`

- Now in terraform code, we should comment the blue node group and change the version to 1.28 manually
