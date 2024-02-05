# Roboshop on Kubernetes (Contd.)

## Redis

- Using the image from Docker Hub directly as we don't have any customisation
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/redis/manifest.yaml)
- We can set alias for the mostly used commands such as: `alias ka="kubectl apply -f manifest.yaml"`

## User

- Similar to catalogue, we fetch the environment variables from `ConfigMap`
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/user/manifest.yaml)
- Once the pods are provisoned, we need to update manifest of web and update the pods

## Cart

- As this is also written in NodeJS, we can follow the same procedure as in Catalogue and User
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/cart/manifest.yaml)
- Once the pods are provisoned, we need to update manifest of web and update the pods

## MySQL

- We can remove the environment variable related information from the Dockerfile as we would like to maintain it using the ConfigMap
- We can encode the secrets and then store them in the ConfigMap for security
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/mysql/manifest.yaml)

## Shipping

- We can remove the environment variable related information from the Dockerfile as we would like to maintain it using the ConfigMap
- There are two kinds of errors that happens in Kubernetes:
  1. Error while pulling the image: This could be due to the unavailability of the image on the docker hub
  2. Configuration error
- [Source Code](https://github.com/sivadevopsdaws74s/k8-roboshop/blob/master/shipping/manifest.yaml)
- Once the pods are provisoned, we need to update manifest of web and update the pods
