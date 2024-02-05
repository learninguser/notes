# Roboshop on Kubernetes (Contd.)

## Rabbitmq

- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/rabbitmq/manifest.yaml)

## Payment

- We don't need to create a separate application user for Rabbitmq
- By default, it uses guest user
- Therefore we don't need to define environment variables for AMQP_USER and AMQP_PASSWORD
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/payment/manifest.yaml)

## Storage on Kubernetes

- So far the data is being stored inside the pods but is not persistent
- Also at any point we increase or decrease the worker nodes, anytime it can be be deleted
- Storing externally is safe
- On AWS, we have Elastic Block Storage (EBS) and Elastic File Storage (EFS) service for the same use which we can mount to any upcoming or existing instances
