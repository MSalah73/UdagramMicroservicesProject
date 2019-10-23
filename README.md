# Udacity - Udagram Microservices Project 

## Project description

Converting the monolithic application Udagram from the previous project into microservices. The conversion process began with decoupling the backend into two services. Then use tools like Docker and Kubernetes to deploy the Udagram Application.  

### Prerequisites
- Install [eksctl](https://docs.aws.amazon.com/eks/latest/userguide/getting-started-eksctl.html)
- Install [awsctl](https://docs.aws.amazon.com/cli/latest/userguide/install-linux.html)
- Install [aws-iam-authenticator](https://docs.aws.amazon.com/eks/latest/userguide/install-aws-iam-authenticator.html)
- Install [kubectl](https://docs.aws.amazon.com/eks/latest/userguide/install-kubectl.html)
- Install [docker](https://docs.docker.com/docker-for-windows/install/)

#### Setup docker enviroment
- Build the images: `docker-compose -f docker-compose-build.yaml  build --parallel`
- Run the containers: `docker-compose up`
- Push the images: `dcoker-compose -f docker-compose-build.yaml push`

#### Creating a cluster with eksctl 
```
eksctl create cluster \ 
--name "ClusterName" \
--version 1.14 \
--nodegroup-name standard-workers \
--node-type t3.medium \
--nodes 3 \
--nodes-min 1 \
--nodes-max 4 \
--node-ami auto
```
#### Modify secrets and configmap .yaml files
- Encrypt the database username and password using base64 using the following commands
    - `echo POSTGRESS_PASSWORD | base64`
    - `echo POSTGRESS_USERNAME | base64`
- Encrypt the aws file using base64 using the following commands
    - `cat ~/.aws/credentials | base64` 
- Add these values in the appropriate places in `env-secret.yaml`, `aws-secret.yaml`, and `env-configmap.yaml` 
#### Setup Kubernetes Environment

- Load secret files: 
	- `kubectl apply -f aws-secret.yaml`
	- `kubectl apply -f env-secret.yaml`
	- `kubectl apply -f env-configmap.yaml`
- Apply all other yaml files:
    - `kubectl apply -f .`

#### Check pods status
-   `kubectl get all`

#### Connect the services with port forwarding
- Use port forwarding to the frontend and reverse proxy services:
    - `Note: The port forwarding must be done in separate terminals, otherwise, it won't work.`
	- `kubectl port-forward service/frontend 8100:8100`
	- `kubectl port-forward service/reverseproxy 8080:

#### CI/CD with TravisCL:
- Sign up for [TravisCL](https://travis-ci.com/) and connect your Github repository to TrivisCL
- Add `.travis.yml` file to the git repository
- Add the following code to the yaml file:
```
language: minimal

services: docker

env:
  - DOCKER_COMPOSE_VERSION=1.23.2

before_install:
  - docker -v && docker-compose -v
  - sudo rm /usr/local/bin/docker-compose
  - curl -L https://github.com/docker/compose/releases/download/${DOCKER_COMPOSE_VERSION}/docker-compose-`uname -s`-`uname -m` > docker-compose
  - chmod +x docker-compose
  - sudo mv docker-compose /usr/local/bin
  - curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
  - chmod +x ./kubectl
  - sudo mv ./kubectl /usr/local/bin/kubectl

install:
  - docker-compose -f udacity-c3-deployment/docker/docker-compose-build.yaml build --parallel 
  ```
  - Add environment variables to the project repository in travis by selecting the setting option.
