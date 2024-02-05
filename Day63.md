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

- So far in Docker and K8s:
  1. Building and pushing the image
  2. Manifest and how to run the image
- When ever there is new version, we can build and release the docker image using CI
- But we need to edit and apply the manifest files manually i.e. enter the latest version information in it
- Generally, we keep the important files aside and edit only required fields i.e. configuration information
- To maintain DRY principle standards, we should be able to pass the required parameters to the function
- To serve this purpose, we use Helm i.e.
  1. Parameterise the K8s manifest files
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
- Next we should have directory named **templates** in which we place the manifest files e.g. deployment.yaml
- To run the helm application: `helm install <release name> .` i.e. **.** indicates the location of Chart.yaml
- For e.g. `helm install nginx .`
- The next important file is: **values.yaml**

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
- How to rollback to previous version: `helm history nginx`, `helm rollback nginx`
- To uninstall a chart: `helm delete nginx`

### 2. Kubernetes Package manager

- Steps involved:
  1. Fetch image
  2. Run manifest
- For e.g. if we want to install EKS EBS CSI drivers in K8s cluster:

  ```bash
  helm repo add aws-ebs-csi-driver https://kubernetes-sigs.github.io/aws-ebs-csi-driver
  helm repo update
  helm upgrade --install aws-ebs-csi-driver \
    --namespace kube-system \
    aws-ebs-csi-driver/aws-ebs-csi-driver
  ```
