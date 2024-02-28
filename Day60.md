# Roboshop on Kubernetes (Contd.)

## Rabbitmq

- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/rabbitmq/manifest.yaml)

## Payment

- We don't need to create a separate application user for Rabbitmq
- By default, it uses guest user
- Therefore we don't need to define environment variables for AMQP_USER and AMQP_PASSWORD
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/payment/manifest.yaml)

## Storage

- Storage is all about performing CRUD operations on the data that is residing in the storage
- The data can be of any type i.e. file, excel, logs, RDBMS, NoSQL etc

### Stateful vs Stateless

- Stateful applications are those which does operations on the data directly i.e. data is very crucial here
- Stateless applications are those which doesn't have any database on its own i.e. no storage is necessary
- If Stateless applications are down, we can restore them easily without impacting the business
- In our Roboshop project:
  - MongoDB, redis, rabbitmq and MySQL components are stateful applications
  - Web, Catalogue, User, Cart, Shipping, Payment and Dispatch are stateless applications

### Current roboshop project scenario

- We clearly know that the pods are ephemeral similarly nodes are also ephemeral
- Therefore the data is being stored inside the pods but is not persistent
- Also at any point we increase or decrease the worker nodes, anytime it can be be deleted
- Therefore we shouldn't store the crucial data of stateful applications inside the pods or nodes
- We should store our data inside an external storage and mount it in the nodes and pods
- On AWS, we have Elastic Block Storage (EBS) and Elastic File Storage (EFS) service for the same use which we can mount to any upcoming or existing instances
