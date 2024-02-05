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
- Since AWS is cloud and EKS is a service on it, AWS cannot allow seperate authorisation for these PaaS models
- Rather we can use AWS own authentication mechanism i.e. IAM and K8s RBAC for authorisation
- Therefore, we need to map IAM and K8s RBAC for proper authentication and authorisation
- For on-premise cluster, both authentication and authorisation can be obtained from EKS cluster

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
  - Role and Role binding only provides access to the namespace but not the cluster
  - To do this, we need to perform clusterRole and clusterRoleBinding
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

## HorizontalPodAutoScaler

- If we specify the number of replicas as 1 or 10, it doesn't scale up or down automatically if the traffic is more or less
- We need to monitor the load on the server such as CPU or memory utlisation
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

- [K9s](https://github.com/derailed/k9s) is a minimalistic terminal UI to interact with your Kubernetes clusters
- To install: `curl -sS https://webinstall.dev/k9s | bash`
- To run: execute `k9s` on terminal
- To access another resource, we can use: `shift + :`
- To view the logs of a pod, press `l` and `esc` to come out
- To test an application's performance i.e. w.r.t. load using a tool called **Apache Benchmark**
- To install, we run: `sudo yum install httpd-tools -y`
- To generate traffic, we can run: `ab -n <no: of requests> -c <no: of concurrent users>`
- We should have [**metric-server**](https://github.com/kubernetes-sigs/metrics-server) installed on K8s cluster to calculate the resource utilisation
- To install, run: `kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml`
- Once installed, we can run: `kubectl top pods -n roboshop`
- **We should always specify the resource limits for all the pods to prevent them consuming all the available resources**
- To implement HorizontalPodAutoscaler, we should already have a deployment and a service

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

- When ever we specify load balancer as a service, K8s creates a **classic load balancer** which is legacy one
- Application Load balancer (ALB) can:
  1. Handle heavy traffic
  2. Has advanced routing mechanisms
- As K8s offers classic load balancer by default, we prefer ingress to route the traffic
- We have 2 kinds of path:
  1. Host path: E.g. amazon.joindevops.online
  2. Context path E.g. joindevops.online/amazon
- To implement ingress, we need to create another resource and install ingress controller in our cluster

  `k8s-ingress/manifest.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: app1
    labels: # these labels belong to deployment
      app: app1
  spec:
    replicas: 1
    selector:
      matchLabels: # these labels should match with pod
        app: app1
    template: # this is the pod definition
      metadata:
        labels:
          app: app1
      spec:
        containers:
        - name: app1
          image: joindevops/app1:v1
          ports:
          - containerPort: 80
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: app1
  spec:
    selector:
      app: app1
    ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80 # this port belongs to service
      targetPort: 80 # this port belongs to container
  ```

- [Ingress Controller reference](https://kubernetes-sigs.github.io/aws-load-balancer-controller/v2.6/)
- Ingress Controllers are installed using Helm charts
- Application Load Balancer (ALB) is external to K8s. Therefore we should use annotations to match the resource
