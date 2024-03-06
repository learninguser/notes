# Improvement in Roboshop implementation in Kubernetes (Contd.)

## Recap

- So far we have discussed the following concepts in K8s:
  - Pod
  - ConfigMap
  - Secret
  - Service
  - Sets
    - ReplicaSet
    - Deployment
    - DaemonSet
    - StatefulSet
  - Static and Dynamic Provisioning
    - Storage Class: Used for Dynamic provisioning
    - PV: Representation of external Storage
    - PVC: A claim to mount the volume
  - StatefulSet vs Deployment
    - StatefulSet is for DB applications. It keeps the network identity same and uses the headless service
    - Deployment is stateless, will not maintain any identity and uses normal service
- For database applications, we should use EBS which is closer to the server so hat the latency is less
- We use Dynamic Provisioning

### Steps involved

1. Install EBS CSI driver
2. Give access to EC2 server
3. Creation of storage class (SC)
    - SC creation is not dependent on application as its an admin activity
    - Therefore it should be outside application manifest files
4. Create PVC and mount to the Pod

## Helm charts

- So far with Docker and K8s:
  1. Build and push the image
  2. Configure the image through manifest files to run them inside pods
- When ever there is new version, we can build and release the docker image using CI
- But we need to edit and apply the manifest files manually i.e. enter the latest version information in it
- Generally, we keep the important files aside and edit only required fields i.e. configuration information
- To maintain DRY principle standards, we should be able to pass the required parameters to the function
- To serve this purpose, we use Helm i.e.
  1. Parameterise (i.e. templatise) the K8s manifest files
  2. Its the Kubernetes Package manager

### Installation of Helm

- [Source](https://helm.sh/docs/intro/install/#from-script)

  ```bash
  curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
  chmod 700 get_helm.sh
  ./get_helm.sh
  ```

- To check if its installed correctly or not: `helm version`

### 1. Parameterise the K8s manifest files

- **Chart.yaml** is a mandatory file that we need to keep as its the metadata of the application

  `Chart.yaml`

  ```yaml
  apiVersion: v1
  name: nginx
  version: 0.0.1 # this the chart version
  description: this is the customised helm chart for nginx
  appVersion: 1.0.1
  ```

- Next we should have directory named **templates** in which we place the manifest files e.g. deployment.yaml
- To run the helm application: `helm install <package-name> .` i.e. **.** indicates the location of Chart.yaml
  - For e.g. `helm install nginx .`
- To list all the packages installed: `helm list`
- Helm in background looks for all the manifest files inside the templates folder and perform `kubectl apply` on them
- The next important file is: **values.yaml** as this is the file where helm looks for the values for the variables

  `values.yaml`

  ```yaml
  deployment:
    imageVersion: latest
    replicaCount: 5
  service:
    port: 80
  ```

  `templates/deployment.yaml`

  ```yaml
  apiVersion: apps/v1
  kind: Deployment
  metadata:
    name: nginx
    labels: # these labels belong to deployment
      app: nginx
  spec:
    replicas: {{ .Values.deployment.replicaCount }}
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
          image: "nginx:{{ .Values.deployment.imageVersion }}"
          ports:
          - containerPort: 80
  ```

  `templates/service.yaml`

  ```yaml
  apiVersion: v1
  kind: Service
  metadata:
    name: nginx-service
  spec:
    type: LoadBalancer
    selector:
      app: nginx
      project: roboshop
      component: frontend
    ports:
    - name: name-of-service-port
      protocol: TCP
      port: {{ .Values.service.port }} # this port belongs to service
      targetPort: http-web-svc # this port belongs to container
  ```

- Once the varaibles are configured, we can upgrade the pods: `helm upgrade nginx .`
  - When performing an upgrade, it is ideal to increase the chart version number in the `Charts.yaml` file i.e. `version: 0.0.2`
- To list all the revisions of a package for e.g. nginx: `helm history nginx`
- How to rollback to previous version: `helm rollback nginx`
  - To rollback to a specific version: `helm rollback nginx <version-number>`
- To uninstall a chart: `helm delete nginx`
- To show the list of parameter values: `helm show values .`
- We can also set values using command line: `helm upgrade nginx --set deployment.replicaCount=3 .`

### 2. Kubernetes Package manager

- Helm has some public repos to configure the image using manifest files
- Steps involved:
  1. Add helm repo using: `helm repo add <repo-name> <repo-url>`
  2. Perform `helm repo update`
  3. Install the application using: `helm upgrade` command
- For e.g. if we want to install EKS EBS CSI drivers in K8s cluster:

  ```bash
  helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
  helm repo update
  helm upgrade --install aws-ebs-csi-driver \
    --namespace kube-system \
    aws-ebs-csi-driver/aws-ebs-csi-driver
  ```

- To uninstall: `helm uninstall <repo-name>`
  - For e.g. `helm uninstall aws-ebs-csi-driver --namespace kube-system` to uninstall aws-ebs-csi-driver
- We search on this [URL](artifacthub.io) for the list of packages that are available and that can be installed using helm
- When ever we get an error that **The connection to the server localhost:8080 was refused** on EKS when trying: `kubectl get nodes` in the workstation, we can use: `aws eks update-kubeconfig --region us-east-1 --name roboshop-cluster` to solve it

### Redis as Helm package

1. Chart.yaml
2. values.yaml
3. templates directory

## K9s UI tool

- It is a minimalistic terminal UI to manage and interact with the Kubernetes resources
- To install: `sudo curl -sS https://webinstall.dev/k9s | bash`
- To switch a resource, type: `shift + :` and then:
  - type `service`: to list all the services
  - type `pods`: to list all the pods in a namespace
  - type `namespaces`: to list all the namespaces
- To view the logs: select the resource and press `l`
- To return from that: press `esc`
- To describe a pod: press `d`
- To delete: `ctrl + d`
- To ignore any existing data, we can pass it as an argument to the pod
  
  ```yaml
  apiVersion: apps/v1
  kind: StatefulSet
  metadata:
    name: mysql
    namespace: roboshop
    labels:
      app: mysql
      project: roboshop
      tier: db
  spec:
    selector:
      matchLabels:
        app: mysql
        project: roboshop
        tier: db
    serviceName: "mysql"
    replicas: {{ .Values.statefulset.replicas }}
    template:
      metadata:
        labels:
          app: mysql
          project: roboshop
          tier: db
      spec:
        containers:
        - name: mysql
          image: "joindevops/mysql:{{ .Values.statefulset.imageVersion }}"
          args:
          - "--ignore-db-dir=lost+found"
          envFrom:
          - configMapRef:
              name: mysql
          - secretRef:
              name: mysql
          volumeMounts:
          - name: mysql
            mountPath: /var/lib/mysql
    volumeClaimTemplates:
    - metadata:
        name: mysql
      spec:
        accessModes: [ "ReadWriteOnce" ]
        storageClassName: "ebs-sc"
        resources:
          requests:
            storage: 2Gi
  ```
