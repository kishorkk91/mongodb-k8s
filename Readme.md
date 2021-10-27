# MongoDB-10.28.1
Deploy bitnami Mongodb on Kubernetes using Helm chart: mongodb-10.28.1

## Introduction
This bitnami mongodb chart allows us to install using two different architecture setups: standalone or replicaset.
Standalone is not good in production environment. Replicaset is good but creates "oplogs" (operations logs) which takes much more diskspace.
If deployment is "standlone" then PVC should be created before deploying of mongodb, if deployment is "replicaset" then PVC will be created automatically.
For current requirement we will use just mongodb as standalone deployment instead of Replicaset.
https://github.com/bitnami/charts/tree/master/bitnami/mongodb

### Setting namespace permanently
`kubectl config set-context --current --namespace=devops-databases
`
### Deploy MongoDB
`helm install mongodb charts/mongodb -f mongodb-custom-values.yaml
`
### How to upgrade prometheus stack instead of uninstall. Ex.
`helm upgrade mongodb charts/mongodb -f mongodb-custom-values.yaml
`
### Additional play around after successful deployment

MongoDB can be accessed on the following DNS name(s) and ports from within your cluster:

    mongodb.devops-databases.svc.cluster.local

To get the root password run:

    export MONGODB_ROOT_PASSWORD=$(kubectl get secret --namespace devops-databases mongodb-root-admin-user -o jsonpath="{.data.mongodb-root-password}" | base64 --decode)

To connect to your database, create a MongoDB&reg; client container:

    kubectl run --namespace devops-databases mongodb-client --rm --tty -i --restart='Never' --env="MONGODB_ROOT_PASSWORD=$MONGODB_ROOT_PASSWORD" --image docker.artifactory.devops.crealogix.com/bitnami/mongodb:4.4.10-debian-10-r0 --command -- bash

Then, run the following command:
    mongo admin --host "mongodb" --authenticationDatabase admin -u root -p $MONGODB_ROOT_PASSWORD

To connect to your database from outside the cluster execute the following commands:

    kubectl port-forward --namespace devops-databases svc/mongodb 27017:27017 &
    mongo --host 127.0.0.1 --authenticationDatabase admin -p $MONGODB_ROOT_PASSWORD


## To deploy MongoDB on cluster, the following docker images are tagged and used from internal Docker repository
####custom mongodb-custom-values.yaml:

repository: bitnami/mongodb
tag: 4.4.10-debian-10-r0

repository: bitnami/nginx
tag: 1.21.3-debian-10-r35

repository: bitnami/bitnami-shell
tag: 10-debian-10-r220

repository: bitnami/mongodb-exporter
tag: 0.11.2-debian-10-r308

### Namespace and PVC
**Namespace:** devops-databases
**PVC:** pvc-mongodb-devops-databases

### Set global parameters if images are pulling from companies/own artifactory
global:
  imageRegistry: "docker.artifactory.devops.MyCompany.com"
  imagePullSecrets:
    - artifactory-docker-regcred
  storageClass: "nfs-client"
  
### In this chart plain text password is used, of course you can use Secrets but just keep it as simple as intially  