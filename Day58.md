# Kubernetes

Date: 07-11-2023

## Provisioning EKS on AWS

- For that, we need a tool from AWS i.e. **eksctl** which is a command line tool
- When we push the manifest files from Workstation to EKS cluster, a node group called for e.g. SpotNG will contain list of servers that will pull the image from Dockerhub
- First we should create an EKS cluster
  - For this, we provision a t2.micro instance with in the default VPC, 30GB of EBS volume and call it as workstation
  - With in the workstation, we install `kubectl`, `eksctl`, `docker` and `kubens`
- We have one master node that is present on EKS and group of worker nodes, we call the group as Spot NG
- The OS that is present on Worker nodes is the choice of AWS. Therefore it chose AWS Linux 2 AMI
- Using `kubens` we can select the K8s namespace
- We can use the following commands to install **kubens**

  ```bash
  sudo git clone https://github.com/ahmetb/kubectx /opt/kubectx
  sudo ln -s /opt/kubectx/kubens /usr/local/bin/kubens
  ```

- We can install all these components using the following script: `curl -O https://raw.githubusercontent.com/sivadevopsdaws74s/kubernetes-eksctl/master/workstation.sh` and use `sudo sh workstation.sh`
- After that we need to configure our workstation instance with a CLI user that has admin rights inorder to provision EKS cluster on AWS
- EKS: Elastic Kubernetes Service from AWS
- We **cannot** have SSH access to the master node and it is managed by AWS itself
- `eksctl` is a simple way to provision an EKS clutser on AWS
- Sample example configs to provision EKS cluster using eksctl can be found [here](https://github.com/eksctl-io/eksctl/tree/main/examples)
- Along with it, we have `eks.yaml` file in which we specify the configuration we would like to have

  `eks.yaml`

  ```yaml
  apiVersion: eksctl.io/v1alpha5
  kind: ClusterConfig

  metadata:
    name: eks-spot-cluster
    region: us-east-1

  # this is also completely managed by AWS
  managedNodeGroups:
    - name: spot
      instanceType: m5.large
      # your K8 node can be anytime taken back by AWS
      spot: true
      desiredCapacity: 3
      ssh:
        publicKeyName: kubernetes # replace this with your keypair that exists in AWS.
  ```

- We specify the number of worker nodes that needs to be a part of the EKS Cluster under `managedNodeGroups`
- Spot instances are not used in production environments rather in development environments and non-crucial workloads
- In case if the spot instance is taken back by AWS, K8s can provision another one immediately
- By default the EKS cluster is provisioned in the Default VPC
- We should generate a keypair with the name kubernetes using `ssh-keygen -f kubernetes` and import the public key on to AWS
- Once everything is ready, then we should run: `eksctl create cluster --config-file=eks.yaml`
- This will take around **15-20** mins of time to provision
- Once the EKS cluster is successfully provisioned, we can run `kubectl get nodes` command on the workstation where eksctl is installeed

## Node port

- To create and attach NodePort service to a pod, we just need to set `type` to `NodePort` under `spec`

  `services/02-nodeport.yaml`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels: # These are used for selecting and filtering the pod
      environment: dev
      app: frontend
      project: roboshop
  spec:
    containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: http-web-svc
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: NodePort
    # search for pods that matches the given labels and attach to it
    selector:
      environment: dev
      app: frontend
      project: roboshop
    ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80 # this port belongs to service
      targetPort: http-web-svc # this port belongs to container
  ```

- Run the command: `kubectl apply -f 02-nodeport.yaml`
- To view all the services: `kubectl get service` or `kubectl get svc`
- Once the service is successfully created, a cluster IP is allocated and random port is also exposed
- A port number in the ephemeral range (30000 - 32767) at random (it will be same for all the nodes) is choosen by K8s on each and every node
- In addition to that a **security group** is also created for all the nodes that in the cluster by EKS on its own
- We should allow the port in the security group that has **remoteAccess** in its name
- Once the port is enabled, we will be able to access the application that is running on the pod using: `<Node-public-ip>:<random-port-number-assigned-by-EKS>`
- **Cluster IP is a subset of Node port** i.e. when we provision a node port service, a cluster IP service is also provisioned in the background
- We can also specify which port to expose using: `nodePort: 30007`

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: NodePort
    # search for pods that matches the given labels and attach to it
    selector:
      environment: dev
      app: frontend
      project: roboshop
    ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80 # this port belongs to service
      targetPort: 80 # this port belongs to container
      nodePort: 30007 # this port belongs to host
  ```

## Loadbalancer

- This service works only for cloud based K8s clusters
- By setting type to LoadBalancer, a loadbalancer service is provisioned
- Similar to Node Port, Load Balancer is created, a random ephemeral port is exposed and cluster IP is also present
- In Node Port, we need to expose the ports on our own for all the nodes that are part of the cluster group
- Also each node has unique public IP linked to it.
- This might be a security concern when we open a range of ports on our server
- To avoid this, we can use a load balancer service from K8s which exposes only the open port, creates a DNS record and routes the traffic to the nodes using Round Robin algorithm

  `services/03-loadbalancer.yaml`

  ```yaml
  apiVersion: v1
  kind: Pod
  metadata:
    name: nginx
    labels: # These are used for selecting and filtering the pod
      environment: dev
      app: frontend
      project: roboshop
  spec:
    containers:
    - name: nginx
      image: nginx
      ports:
        - containerPort: 80
          name: http-web-svc
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: LoadBalancer
    # search for pods that matches the given labels and attach to it
    selector:
      environment: dev
      app: frontend
      project: roboshop
    ports:
    - name: name-of-service-port
      protocol: TCP
      port: 80 # this port belongs to service
      targetPort: http-web-svc # this port belongs to container
  ```

- All the node group instances are added as a part of the LB under Target Instances

### Flow of a request

- Once a user sends a request using the load balancer address
  - Step 1: LB transfers it to any of the node in the cluster
  - Step 2: Then its forwarded to the Cluster IP
  - Step 3: Finally it reaches the pod where the App is hosted
- We can also create an Alias record on Route53 to the Load Balancer to keep its address short

## Sets in K8s

1. Replica set
2. Deployment set
3. Daemon set
4. Stateful set

### 1. Replica set

- Suppose if the traffic is very high, we need to change the name of the pod manually and provision it manually which is a painful process
- Instead we can handover this job to K8s using ReplicaSet and it provisions the pods on our behalf

  `sets/01-replicaset.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: ReplicaSet
  metadata:
    name: frontend
    labels: # these labels are replica set labels, every k8 resource can have labels.
      app: nginx
      tier: frontend
  spec:
    # modify replicas according to your case
    replicas: 3
    selector:
      matchLabels: # these labels should match with pod labels
        app: nginx
        tier: frontend
        environment: dev
    template: # this one is nothing but pod definition.
      metadata:
        labels: # these are the pod labels.
          app: nginx
          tier: frontend
          environment: dev
      spec:
        containers:
        - name: nginx
          image: nginx:1.14.2
  ```

- ReplicaSet is present in `apps/v1` resource type. Therefore, we set `apiVersion` to `apps/v1`
- `kubectl get rs` to check the status of the replicasets
- A replicaset creates pods internally
- **If there is an update in the image, it will not update the exisiting pods in the replicaset** since its main purpose is to provide the no: of replicas mentioned in the manifest file
- Therefore we go for **deployment set**

### 2. Deployment Set

- This is the most commonly used among others

  `sets/02-deployment.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx-deployment
    labels: # these labels belong to deployment
      app: nginx
  spec:
    replicas: 3
    selector:
      matchLabels: # these labels should match with pod
        app: nginx
        project: roboshop
        component: frontend
    template: # this is the pod definition
      metadata:
        labels:
          app: nginx
          project: roboshop
          component: frontend
      spec:
        containers:
        - name: nginx
          image: nginx
          ports:
          - containerPort: 80
  ```

- We can get all the deployments using: `kubectl get deploy`
- When we create a Deployment, it creates a Replicaset with the `replicaSet_name-random_ID`
- Deployment created another replicaset, therefore Replicaset is a subset of deployment
- Therefore, when we check: `kubectl get rs` to get the list of ReplicaSets, we can see it has the name: `deployment_name-random_ID`
- Using this, **we can acheive Zero Downtime** of our application

### How does it achieve it?

- When an update takes place, Deployment creates a new replicaset and uses the updated image when creating the pods
- Once a pod is provisioned, it terminates the pod from previous replicaset that is outdated as it needs to maintain the number of replicas

## Roboshop on EKS

- Broadly there are 2 steps that are involved:
  1. Image should exist i.e. build the image
  2. Configure the image using manifest files

### Common errors when implementing project with K8s

1. Either image exists or not ?
2. Is the image properly built ?
3. Did we specify the version of the image correctly?
4. Does the K8s cluster has access to pull the images from docker hub?
5. If everything is proper, is the configuration information specified is correct or not?

### Setup flow

1. Develop Dockerfiles and Manifest files locally and push them to GitHub
2. Pull them into Workstation and build the Dockerfile
3. Push the images to Docker Hub
4. Push the manifests to EKS Cluster

### MongoDB

- We start by creating a namespace

  `namespace.yaml`

  ```yaml
  apiVersion: v1
  kind: Namespace
  metadata:
    name: roboshop
  ```

- Within the manifest files, we specify our deployments

  `mongodb/manifest.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mongodb
    namespace: roboshop
    labels: # these labels belong to deployment
      app: mongodb
      tier: db
      project: roboshop
  spec:
    replicas: 1
    selector:
      matchLabels: # these labels should match with pod
        app: mongodb
        tier: db
        project: roboshop
    template: # this is the pod definition
      metadata:
        namespace: roboshop
        labels:
          app: mongodb
          tier: db
          project: roboshop
      spec:
        containers:
        - name: mongodb
          image: joindevops/mongodb:1.0.0
  ```

#### Commands

1. `kubectl apply -f namespace.yaml`
2. `docker build -t joindevops/mongodb:1.0.0 .`
3. `docker login`
4. `docker push joindevops/mongodb:1.0.0`
5. `kubectl apply -f manifest.yaml`
6. `kubens roboshop`

- Using the 6th command, we don't need to mention `-n roboshop` everytime to fetch the resources provisioned in that namespace
- To confirm if the pood is created or not, we can run: `kubectl get pods -o wide`
- Now catalogue should connect with the MongoDB
- Connecting to it using IP address is not safe
- Therefore we use a service and as we don't need to expose the MongoDB port to the outside world, we can just use: **Cluster IP** service
- **Note**: A pod should always have a service attached to it in order to enable other pods to communicate with it
- **Attaching Cluster IP service to MongoDB**

  `mongodb/manifest.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: mongodb
    namespace: roboshop
    labels: # these labels belong to deployment
      app: mongodb
      tier: db
      project: roboshop
  spec:
    replicas: 1
    selector:
      matchLabels: # these labels should match with pod
        app: mongodb
        tier: db
        project: roboshop
    template: # this is the pod definition
      metadata:
        labels:
          app: mongodb
          tier: db
          project: roboshop
      spec:
        containers:
        - name: mongodb
          image: joindevops/mongodb:1.0.0
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: mongodb
    namespace: roboshop
  spec:
    selector:
      app: mongodb
      tier: db
      project: roboshop
    ports:
    - name: mongodb-port
      protocol: TCP
      port: 27017 # this port belongs to service
      targetPort: 27017 # this port belongs to container
  ```

### Catalogue

- As we don't need to expose the catalogue port to external users, we can use **ClusterIP** service
- When using Docker with K8s, we should avoid storing variables as environment variables at the docker image layer
- Rather, we can define these environment variables using **ConfigMap**, therefore removing it from the Dockerfile
- Whenever there is a change in the application code, only then we should go for CICD process
- Therefore, we should maintain the configuration information outside the application code

  `catalogue/manifest.yaml`

  ```yaml
  apiVersion: v1
  kind: ConfigMap
  metadata:
    name: catalogue
    namespace: roboshop
  data:
    MONGO: "true" # keep true in double quotes
    MONGO_URL: "mongodb://mongodb:27017/catalogue" # protocol//service-name:service-port/schema
  ---
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: catalogue
    namespace: roboshop
    labels: # these labels belong to deployment
      app: catalogue
      tier: app
      project: roboshop
  spec:
    replicas: 1
    selector:
      matchLabels: # these labels should match with pod
        app: catalogue
        tier: app
        project: roboshop
    template: # this is the pod definition
      metadata:
        namespace: roboshop
        labels:
          app: catalogue
          tier: app
          project: roboshop
      spec:
        containers:
        - name: catalogue
          image: joindevops/catalogue:1.0.0
          envFrom:
          - configMapRef:
              name: catalogue
  ---
  apiVersion: v1
  kind: Service
  metadata:
    name: catalogue
    namespace: roboshop
  spec:
    selector:
      app: catalogue
      tier: app
      project: roboshop
    ports:
    - name: catalogue-port
      protocol: TCP
      port: 8080 # this port belongs to service
      targetPort: 8080 # this port belongs to container
  ```

- To provision the resources: `kubectl apply -f manifest.yaml`
- To check the logs:

  ```bash
  kubectl get pods
  kubectl logs <Name-of-the-pod>
  ```

- To debug the pods that are created, we can use a sample container with an image for e.g. almalinux that has basic commands such as telnet
- There can also be a case where the pods are being created in a different node i.e. mongo in one node, catalogue in another node
- Due to this, catalogue pod will not be able to communicate with the mongo pod

### Web Component

- Build is a costly operation, therefore we should:
  - Perform build and restart only when we have a code change
  - Perform restart only when there is a configuration
- Therefore we store the config information inside the ConfigMap as it allows to import config files as well
- [Source code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/da90c171f7b97ae1afbd596089f9130f930a287c/web/manifest.yaml)
- Once we are done with our practice, we should clean up all the resources using:
  - `kubectl delete namespace roboshop` which also deletes the resources provisioned in that namespace
  - `kubens default`: Switching back to the default namespace
  - Or delete the whole cluster: `eksctl delete cluster --config-file=eks.yaml`
- `eksctl` uses cloudformation in the background which is native to AWS
