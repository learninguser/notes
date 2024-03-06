# RBAC (Role based Access control)

- In any system, Authentication and Authorisation is very important
- So far we have been doing everything with Administrator access but it is not possible in the realtime projects
- Therefore we should have Roles and Permissions
- There are 4 objects:
  1. Role
  2. RoleBinding
  3. ClusterRole
  4. ClusterRoleBinding
- Resources in K8s are classified in namespace and cluster level
  - Pod: Namespace
  - Service: Namespace
  - Deployment: Namespace
  - Stateful: Namespace
  - PV: ClusterLevel
  - PVC: Namespace
  - ConfigMap: Namespace
  - nodes: ClusterLevel
- We can view the full list: `kubectl api-resources`
- We can have any number of projects that can have their resources on a single EKS cluster
- EKS is a PaaS model not an AWS native service. The original service is Kubernetes
- PaaS model has their own set rules and regulations
- EKS admins have full access to the entire cluster
- In Roboshop project:
  - roboshop-admin: DevOps and cloud engineers
  - roboshop-developers: Developers with limited access
- EKS Admin will create a namespace
- So far whatever the commands we executed, we did it using admin privileges
- Since AWS is cloud and EKS is a service on it, AWS cannot allow seperate authorisation for these PaaS models
- Rather we can use AWS own authentication mechanism i.e. IAM and K8s RBAC for authorisation
- Therefore, we need to map IAM and K8s RBAC for proper authentication and authorisation
- For on-premise cluster, both authentication and authorisation can be obtained from EKS cluster
- There are a few objects in K8s that are specific to RBAC:
  1. Role
  2. RoleBinding
  3. ClusterRole
  4. ClusterRoleBinding

- It is similar to an IAM user on AWS to whom a Role is attached or some policies are attached to it

- **Step 1: Authentication**
  - We create a new IAM user and provide access to the cluster in order to describe it i.e. read it
  - To provide access to the user, we create and attach a policy to it

- **Step 2: Role and Role Binding**
  - Each role has permissions attached to it
  - For e.g. In an organisation
    - trainee has read permission
    - junior engineer
    - senior engineer
    - team leader has module admin
    - manager has entire project admin
    - MD has account admin
- Here we have created a namespace, role and its permissions

  `k8-rbac/admin.yaml`

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: roboshop
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: Role
  metadata:
    namespace: roboshop
    name: roboshop-admin
  rules:
  - apiGroups: ["*"] # "" indicates the core API group
    resources: ["*"]
    verbs: ["*"] # actions on resources, create, read, update, delete
  ```
  
  - Once the role is created, we need to bind this role to a user

  `k8s-rbac/admin.yaml`

  ```yaml
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: RoleBinding
  metadata:
    name: roboshop-admin
    namespace: roboshop
  subjects:
  # You can specify more than one "subject"
  - kind: User
    name: ramesh # "name" is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    # "roleRef" specifies the binding to a Role / ClusterRole
    kind: Role #this must be Role or ClusterRole
    name: roboshop-admin # this must match the name of the Role or ClusterRole you wish to bind to
    apiGroup: rbac.authorization.k8s.io
  ```

  - Once a role is created and binded to a user, we need to inform EKS about it
  - When an EKS cluster is created, there is a configmap called **aws-auth** in **kube-system** namespace
  - In this, we need to map the IAM user with K8s-RBAC

- **Step 3: Inform aws-auth configmap**
  - We can get the kube-system configmap information with the user what ever is created by executing the below command:
    - `kubectl get configmap aws-auth -n kube-system -o yaml` where `-o` indicates output format
  - With in the aws-auth, we have mapRoles already. We just need to map users to roles

  `k8s-rbac/auth.yaml`

  ```yaml
  apiVersion: v1
  data:
    mapRoles: |
      - groups:
        - system:bootstrappers
        - system:nodes
        rolearn: arn:aws:iam::315069654700:role/eksctl-eks-spot-cluster-nodegroup--NodeInstanceRole-ZFRyOLy0FVrN
        username: system:node:{{EC2PrivateDNSName}}
    mapUsers: |
      - groups: # Role
        - roboshop-admin
        userarn: arn:aws:iam::315069654700:user/ramesh
        username: ramesh
  kind: ConfigMap
  metadata:
    creationTimestamp: "2023-11-21T01:43:14Z"
    name: aws-auth
    namespace: kube-system
    resourceVersion: "8642"
    uid: 94eacb41-7c0a-4038-91cf-d6ad800e1867
  ```

- For newly created user, we need to also have CLI credentials
- So far, we are acting as an EKS cluster admin. Now, we create a new EC2 instance and acts as a roboshop admin to see if the newly created user i.e Ramesh can only access resources within the roboshop namespace
- To access the cluster:
  - Check whether the instance is configured with a user or not, we can use: `aws sts get-caller-identity`
  - To create and update `.kubeconfig`, we run: `aws eks update-kubeconfig --region us-east-1 --name eks-spot-cluster`
  - Install `kubectl` and its dependencies on this newly created instance using: `curl -L https://raw.githubusercontent.com/sivadevopsdaws74s/kubernetes-eksctl/master/workstation.sh | sudo bash`
  - Once they're installed, we can check whether we can list the nodes: `kubectl get nodes`
  - Nodes is a cluster resource and inside nodes we have namespaces
  - **Role and Role binding can be used for namespace level resources only**
  - Whereas to provide access to **Cluster level resources**, we use **clusterRole and clusterRoleBinding**
  - Also, we restrict to limited access such as list of nodes available, list of namespaces available etc but not PV creation etc

- **Step 4: ClusterRole and ClusterRoleBinding**
  - Cluster role creation
  - To get the list of resources and groups available: `kubectl api-resources`

  `k8s-rbac/admin.yaml`

  ```yaml
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRole
  metadata:
    # "namespace" omitted since ClusterRoles are not namespaced
    name: roboshop-cluster-reader
  rules:
  - apiGroups: ["v1", "apps/v1", ""] # "" indicates the core API group
    resources: ["secrets", "nodes", "namespaces", "persistentvolumes"]
    verbs: ["get", "watch", "list"] # Restricted to Read access only.
  ```

  - ClusterRoleBinding creation

  `k8s-rbac/admin.yaml`

  ```yaml
  ---
  apiVersion: rbac.authorization.k8s.io/v1
  kind: ClusterRoleBinding
  metadata:
    name: roboshop-cluster-reader
  subjects:
  - kind: User
    name: ramesh # Name is case sensitive
    apiGroup: rbac.authorization.k8s.io
  roleRef:
    kind: ClusterRole
    name: roboshop-cluster-reader
    apiGroup: rbac.authorization.k8s.io
  ```

- For developers, we don't need to provide ClusterRole and ClusterRoleBinding

## HorizontalPodAutoScaler (HPA)

- If we specify the number of replicas as 1 or 10, it doesn't scale up or down automatically if the traffic is more or less
- To do this, we need to use HPA and monitor the load on the server such as CPU or memory utilisation
- To monitor the load on the server dynamically, we should install **Metrics API server** on K8s cluster
- [**metrics-server**](https://github.com/kubernetes-sigs/metrics-server)
- To install, run: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
- Once installed, we can run: `kubectl top pods -n roboshop`
- **We should always specify the resource limits for all the pods to prevent them consuming all the available resources**
- To implement HorizontalPodAutoscaler, we should already have a deployment and a service
- To deploy all the resources in one go:
  
  `k8-roboshop`

  ```bash
  services=(mongodb redis mysql rabbitmq catalogue cart user shipping payment web)
  for i in "${services[@]}"
  do
    cd $i
    kubectl apply -f manifest.yaml -n roboshop
    cd ..
  done
  ```

  `k8s-roboshop/web/manifest.yaml`

  ```yaml
  ---
  apiVersion: autoscaling/v2
  kind: HorizontalPodAutoscaler
  metadata:
    name: web
    namespace: roboshop
  spec:
    scaleTargetRef:
      apiVersion: apps/v1
      kind: Deployment
      name: web
    minReplicas: 1
    maxReplicas: 10
    metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 75
  ```

- **Databases use High memory**. Therefore we shouldn't keep them inside K8s

## Ingress

- When ever we specify load balancer as a service, K8s creates a **classic load balancer** which is a legacy one
- Application Load balancer (ALB) can:
  1. Handle heavy traffic
  2. Has advanced routing mechanisms
- As K8s offers classic load balancer by default, we prefer an **ingress controller** to route the traffic
- We have 2 kinds of path:
  1. Host path: E.g. amazon.joindevops.online
  2. Context path E.g. joindevops.online/amazon
- When ever a request is sent to **app1.daws76s.online**, the request is first forwarded to ingress controller -> ingress -> app1 service -> app1 pod
- Usually, a target group is attached to the laod balancer. This can be done using ingress without touching the load balancer.
- EKS should have the capability to create load balancers, target groups, listeners, rules etc
- To implement this, we should install ingress controller in our cluster

### Steps to install ingress controller in K8s cluster

- The steps are listed in [Ingress Controller reference](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.7/)

1. Create and associate IAM OIDC (Open ID connect) provider for authentication purpose with the resources that are external to EKS
2. Download IAM policy for the LBC using the commands
3. Create a policy using the file obtained in step 2
4. Create an IAM role and K8s ServiceAccount for the LBAC
5. Ingress Controllers are installed using Helm charts

  `k8s-ingress/manifest.yaml`

  ```yaml
  apiVersion: networking.k8s.io/v1
  kind: Ingress
  metadata:
    name: app1
    annotations:
        alb.ingress.kubernetes.io/scheme: internet-facing
        alb.ingress.kubernetes.io/target-type: ip
        alb.ingress.kubernetes.io/tags: Environment=dev,Team=test,Project=Roboshop
        alb.ingress.kubernetes.io/group.name: joindevops
  spec:
    ingressClassName: alb
    rules:
    - host: "app1.learninguser.shop"
      http:
        paths:
        - pathType: Prefix
          path: "/"
          backend:
            service:
              name: app1
              port:
                number: 80
  ```

- Recall, labels are used for selecting K8s objects where as annotations are used to match the resources that are external to K8s
- Application Load Balancer (ALB) is external to K8s. Therefore we should use annotations to match the resource
- Each ingress service will create a load balancer on its own but in our case, we wanted to have one load balancer and route traffic to a resource based on the url context path
- For this, we will use another annotation: `alb.ingress.kubernetes.io/group.name` and specify a group name to it
- This will restrict in creating another load balancer for the same group if exists
- We need to add `EC2FullAccess` policy to the IAM that is attached to the worker nodes inorder to create the loadbalancer

## Apache Benchmark tool

- To test an application's performance i.e. w.r.t. load, we can use a tool called **Apache Benchmark**
- To install, we run: `sudo yum install httpd-tools -y`
- To generate traffic, we can run: `ab -n <no: of requests> -c <no: of concurrent users>`

## K8s architecture

- A system architecture is nothing but understanding how it does it accept and processes an incoming request and sends it output using outgoing requests
- K8s is also based on Master and worker node concept
- Here, the master node is the control plane in EKS cluster
- Both control plane and worker nodes have some components

### Control Plane components

- It is the responsibility of the cloud vendors to manage the control plane

1. Kubectl: Its a command line tool to interact with the kubernetes cluster
2. kube-apiserver: It exposes the K8s API and responds to the kubectl commands
      - When we issue `kubectl get pods` command, first `kubectl` communciates with the kube-apiserver
      - It will perform validation of the request i.e. performs authentication and authorisation as it is the **front-face** of a Kubernetes cluster
3. kube-scheduler: Creation of resources is handled by Scheduler
      - When we issue `kubectl apply -f mainfest.yaml`, the request is forwarded from apiserver to the scheduler
      - This decides and schedules the tasks in the worker nodes for the pods creation based on few factors
      - It also takes few factors into consideration such as node labels, selectors, taints and tolerations, affinity and non-affinity
4. etcd:
      - This stores entire data such as configmaps, secrets etc related to the K8s cluster
      - It is very crucial to store this data securely and if its lost, we can't retrive the information related to the resources that're provisioned by K8s
      - Therefore, we keep it outside K8s cluster i.e. use some external storage options such as EBS or EFS
      - Usually if we opt for cloud based K8s cluster, it is their responsibility to take regular backups of this database
5. kube-control-manager:
      - Its responsibility is to run different controller processes
      - There are different types controllers such as:
        - Node controller: It is responsible to inform the health of a node and act accordingly
        - Job controller: It controls the pod objects that runs a particular job and then exits
        - Replication controller: It monitors the requested no: of replicas are running properly all the time

### Worker node components

1. kubelet: It makes sure that the worker nodes are connected to the control plane
2. kube-proxy: Its responsible to forward the request from services to the correct pods in the worker node
3. Container runtime: Its a general runtime that can run any images into containers

## Deployment strategies

- Rolling update: Zero downtime strategy
  - Disadvantage: At a same time the application is serving both older and newer versions i.e for frew seconds
- Blue/green: Blue is the running version and green is the new version
  - 4 current pods are running i.e. current version, 4 new set of pods will run i.e. new version
  - We will perfom some sanity checks on the current version such as health checks and if everything is fine, then we change the DNS records
  - Lets say we have a target group that points to the older version and another that points to th newer version
  - We update the rule to send requests to the target group linked with new version
  - This will be tested and receive feedback from the customers who used it
  - If there is an issue with the latest version, they will revert back to the previous working target group
  - If no issues, then the latest version will be released to all the customers i.e. green becomes blue
  - Disadvantage: Expensive as we need to maintain double resources for sometime and proper testing is necessary before releasing it to public
- Canary: Version B is released to a certain subset of users and then proceeded to a full rollout
  - Disadvantage: Slow rollout
- With Recreate strategy there is downtime that is expected i.e. a downtime is announced then the v1 application is completely taken down, v2 is deployed and the traffic is directed to the new version

## EKS Upgrade using Blue Green Deployment strategy

- Lets say we have 3 nodes that are running on K8s v1.27
- Now we wish to upgrade them to v1.29 of K8s
- **If control plane is down, the applications will work**
- First, we should upgrade the master node
- When performing the upgrade, the master node cannot accept requests for creating new resources
- But the applications that are deployed earlier will keep running but we cannot modify them
- To be on safe side, we announce a down time as its a platform upgrade whereas for application upgrade there is no need to announce
- We will create another node group for v1.29 and slowly drain the nodes i.e. for e.g. old-node-1 is drained and other old nodes are tainted
- Drain: Delete all the pods from the node and restrict them not to go to the other old nodes
- Then pods will be automatically pushed into new nodes
- Once all the old nodes are drained and pods are deployed in the new nodes, we will delete old node group
- In this approach, old node group is blue and new node group is green
- Now in future, when v1.30 is available, new node group will become blue and old node group with v1.29 will be considered as blue
- This way blue and green are toggled
