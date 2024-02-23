# Docker Continued

## Layers

- Once a docker image is created, we cannot modify it externally
- But within the Docker environment we can modify it using Docker commands
- Each instruction in the Dockerfile is a layer on its own
- At each layer a container is created by the Docker Engine
- And within the each layer i.e. container, further instructions are appended one-by-one
- With each appended instruction and executed, a new layer is created
- Once a container is created from that layer, the image corresponding to that layer is automatically deleted
- As the number of instructions in the Dockerfile increases, the build time also increases
- To reduce this time, docker caches the existing layers at the time of build and uses it when we rebuild the image
- If we can combine and optimize the layers such as RUN instructions, the size of the image and its build time can reduce
- It is always recommended to have instructions that are frequently changing at the end of the Dockerfile to reduce build time

## Disadvantages

- What happens if the Docker server is crashed?
  - Application goes down + its associated data is also lost
- What happens if we have sudden increase/decrease in traffic?
  - We have to run multiple containers manually
- How to balance the load if we have multiple containers for a component e.g. cart ?
  - How to split the traffic?
- Self Healing: How to handle a situation when a container crashes
  - Create a new container
- How to handle configs and secrets?
  - No environment to store secrets and configs
- What if we have multiple host running with containers? i.e. how can they communicate with them?

## Container Orchestration

- To overcome above disadvantages, we can use an orchestrator
- We need to someone to give instructions to the container based on the live traffic
- E.g. a manager at a company who handles different teams and instructs each team what should they work on i.e. distributes work to the corresponding teams for a better productivity and manages us
- Similarly we need some orchestration tool to monitor the containers or give instructions at runtime based on the live traffic
- Out of all available tools, Kubernetes is popular. Docker Swarm is also present but has lot of challenges and is not scalable
- We still build our images using Docker and hand over the container management lifecycle to the Kubernetes
- Kubernetes (K8s) offers better networking and storage solutions
- Every cloud offers K8s as solution but not docker swarm

## Kubernetes installation

- Kubernetes is a cluster and offers PaaS (Platform as a Service) model
- Kubernetes is a cloud agnostic service i.e. it can be used on any cloud platform, on-prem etc
- Every clustering solution has a Master and Worker nodes
  - Master node: Assign instructions to worker nodes
  - Worker nodes: Performs task based on the instructions from Master
- If there are no worker nodes, Master node needs to perform all the tasks
- In production grade or high-end application, we have more than one replica of master node (control plane). Usually 3 is a good number
- **Minikube**: Installing all the Master and worker components within **one** server
- But this minikube is only for learning purpose and not production ready
- Manifest files are written in yaml and is a way to inform K8s to run the image
- `kubectl` is the command line that connects with the Kubernetes cluster and pushes the manifest files
- We pass on the manifest file to control plane and it will ask its worker to perform the task
- We only talk to control through the API to the control plane and don't talk to the worker nodes directly
- Everything will only happen on the worker nodes
- K8 cluster pulls the images from Dockerhub and creates containers which are referred as **Pods** in Kubernetes
- A pod is the smallest unit in K8s
- If Minikube is successfully installed, a file with name **kubeconfig** will be created in our home directory
- We should create a directory called `.kube` and copy `kubeconfig` file as `config` to `.kube` directory
- After that if we run `kubectl get nodes` command, we should see a node listed in the output
- Everything in Kubernetes is referred as a resource

## Kubernetes Resources

### Namespace

- An isolated project space
- With in this, we create project resources
- To view all the namespaces: `kubectl get namespaces`
- If we don't specify our own namespace, K8s will use the default namespace
- The other namespaces are used by Kubernetes cluster internally

#### Creating a new namespace

`namespace.yaml`

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: roboshop
```

- To create a namespace: `kubectl create -f 01-namespace.yaml`
  - `-f` - filename
- With the above command, kubectl connects to the cluster and creates the namespace
- If we try to apply the same command once again, we get an error
- Difference between `kubectl create` and `kubectl apply`:
  - `kubectl create`: If a resource doesn't exist, it creates and if exists we get an error
  - `kubectl apply`: When resource doesn't exist, it will create and if exists it updates or doesn't do anything
- Therefore using `kubectl apply -f <filename>.yaml` is better than `kubectl create`
- To delete a namespace: `kubectl delete -f <filename>.yaml`
- `apiVersion` is used for categorizing the resources in kubernetes
- We can get the list of resources using: `kubectl api-resources`
