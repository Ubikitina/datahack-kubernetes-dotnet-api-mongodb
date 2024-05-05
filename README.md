# Docker and Kubernetes Deployment of MongoDB-Backed API for Car Listings

## Table of contents
- [Docker and Kubernetes Deployment of MongoDB-Backed API for Car Listings](#docker-and-kubernetes-deployment-of-mongodb-backed-api-for-car-listings)
  - [Table of contents](#table-of-contents)
  - [Objective](#objective)
  - [Solution Architecture](#solution-architecture)
  - [Docker Compose demo](#docker-compose-demo)
    - [Definition and Creation of Images](#definition-and-creation-of-images)
    - [Creation of Version v2 of the Image](#creation-of-version-v2-of-the-image)
    - [Publishing Images to a Container Registry](#publishing-images-to-a-container-registry)
    - [Deployment of the Solution](#deployment-of-the-solution)
    - [Execution of Commands within a Container](#execution-of-commands-within-a-container)
    - [Using the Mongo Express Web Interface](#using-the-mongo-express-web-interface)
    - [Database Persistence](#database-persistence)
    - [Configuring Resource Usage of the Solution (Memory and CPU)](#configuring-resource-usage-of-the-solution-memory-and-cpu)
    - [Scaling the Solution](#scaling-the-solution)
    - [Uninstalling the Solution and Cleaning Up from our PC](#uninstalling-the-solution-and-cleaning-up-from-our-pc)
  - [Kubernetes demo](#kubernetes-demo)
    - [Solution Design](#solution-design)
    - [Solution Deployment, Running Commands within a Container, and Using the Web Interface](#solution-deployment-running-commands-within-a-container-and-using-the-web-interface)
    - [Database Persistence](#database-persistence-1)
    - [Resource Usage Configuration](#resource-usage-configuration)
    - [Scaling the Solution](#scaling-the-solution-1)
    - [Uninstalling the Solution and Removing Everything from Your PC](#uninstalling-the-solution-and-removing-everything-from-your-pc)
  - [Improvements for Future Versions](#improvements-for-future-versions)
    - [web-api.yaml File](#web-apiyaml-file)
    - [Deployment vs StatefulSet for MongoDB Persistence](#deployment-vs-statefulset-for-mongodb-persistence)
    - [Rolling Update for the Web API Service](#rolling-update-for-the-web-api-service)


## Objective

The objective of this project is to deploy, both in a local Docker Compose environment and in a Kubernetes environment, two versions of an API that lists cars from a MongoDB database. The solution comprises three images: one for the database (MongoDB), one for its web interface (Mongo-express) and another where a web solution needs to be compiled, which queries the database and exposes that information through an API accessible from a browse.

## Solution Architecture

For the implementation of this practice, three images have been implemented:

1. MongoDB database.
2. Web API developed in .NET that queries the database and exposes the information. The information exposed by the API includes:
   - The API version.
   - A value from the web application environment.
   - The list of cars from the database.
3. Mongo-express: A minimalist and user-friendly web interface for managing MongoDB databases.

## Docker Compose demo

### Definition and Creation of Images

We create the first image of the web API: `docker build -t web_api .`

Listing the images: `docker images`

Since the created image is the first version, we tag it as v1: `docker tag web_api:latest web_api:v1`

Removing the latest tag to leave only v1: `docker rmi web_api:latest`

### Creation of Version v2 of the Image

To create the new version v2 of the image, we change line 20 of the PracticaController.cs file.

Now we build it, in this case, directly with v2 set: `docker build -t web_api:v2 .`

We list the images to see that we have both, v1 and v2: `docker images`

### Publishing Images to a Container Registry

To publish to my DockerHub, first, we have to login: `docker login`

Now let's tag our image with my Dockerhub username, both v1 and v2: 
```
docker tag web_api:v1 [username]/web_api:v1 
docker tag web_api:v2 [username]/web_api:v2
```

Now we can publish it by pushing:
```
docker push [username]/web_api:v1 
docker push [username]/web_api:v2
```

We check at https://hub.docker.com/ to verify that the images are there.

Now we delete the images from my local repository: 
```
docker rmi web_api:v1 
docker rmi web_api:v2 
docker rmi [username]/web_api:v1 
docker rmi [username]/web_api:v2
```

We verify that they have been deleted: `docker images`

### Deployment of the Solution

First, we will test the deployment with version v1. Put v1 in the docker-compose.yaml file.

```
docker-compose up -d
```

To see which port it is running on and verify that the API returns an empty list:
```
docker-compose ps
```
Stop and remove containers:
```
docker-compose down
```

Now we will test the deployment with version v2. Put v2 in the docker-compose.yaml file and repeat the commands above. Similarly, we can check that it is running in the browser and returns an empty list. We can also check that the API now indicates version 2 in the displayed data.

### Execution of Commands within a Container

Connect to the terminal of a MongoDB instance running in a Docker container: 
```
docker exec -it mongodb-container mongo
```

Test that the connection to the terminal has been successful: 
```
> db.version() 4.0.7
```

As we have enabled authentication, to insert records, first, we need to authenticate.

Check that without authenticating we cannot do anything: 
```
use testdb 
> db.cars.insert({name: "Audi", price: 52642})
```

Authenticate ourselves, for this: 
```
use admin db.auth("root", "password")
```

Now we can insert records. To do this, we will create a database called testdb, just by starting to use it: `use testdb`. Check that we are in the correct database: `db`.

Insert records into the database: 
```
> db.cars.insert({name: "Audi", price: 52642}) 
> db.cars.insert({name: "Mercedes", price: 57127}) 
> db.cars.insert({name: "Skoda", price: 9000}) 
> db.cars.insert({name: "Volvo", price: 29000}) 
> db.cars.insert({name: "Bentley", price: 350000}) 
> db.cars.insert({name: "Citroen", price: 21000}) 
> db.cars.insert({name: "Hummer", price: 41400}) 
> db.cars.insert({name: "Volkswagen", price: 21600})
```

Check what our collection is called:

```
db.getCollectionNames() [ "cars" ]
```

Look at the contents of our collection in a readable way: 
```
db.cars.find().pretty()
```

Exit the terminal: `exit`

Check in the API that the inserted records are visible.

### Using the Mongo Express Web Interface

Access http://localhost:8081/. The first time it will ask for a username and password, which are "admin" and "pass".

Click on `tesdb`, and then on the button to view the `cars` collection. There, we can see the records we have included.

Add new records by clicking on `New Document`, and typing in the details.

Also, try deleting, and verify that the API returns the new added record and has removed the deleted ones.

### Database Persistence

To demonstrate that data persistence has been implemented, we remove the containers: 
```
docker-compose down
```

Check that they are not present: 
```
docker container ls -a
```

We then bring them up again: 
```
docker-compose up -d
```

And verify that the API endpoint in the browser still returns all the data.

We can also try to enter the database: `docker exec -it mongodb-container mongo`

In the Mongo DB console, we authenticate and execute the following: 
```
use admin 
db.auth("root", "password") 
use testdb 
db.cars.find().pretty()
```
Exit the Mongo DB console: `exit`

We can also check the same in the Mongo Express user interface: http://localhost:8081/.

### Configuring Resource Usage of the Solution (Memory and CPU)

To review if such configuration has been applied, we will execute the following commands: 
```
docker stats
docker inspect <container_id_or_name>
```
It prints all details of our container. Here we need to review within the `HostConfig`, the values:

- Memory: The memory is configured at 536870912 bytes, which is approximately 512 megabytes.
- NanoCpus: NanoCpus are 500000000, which corresponds to limiting the container to 50% of a single CPU core (since 500000000 nanoseconds is half the time of a CPU core in one second).

We can directly access these values by: 
```
docker inspect --format='{{ json .HostConfig.Memory }}' 7703dba6c9a7
docker inspect --format='{{ json .HostConfig.NanoCpus }}' 7703dba6c9a7
```

### Scaling the Solution

With docker-compose, we can scale the web solution using commands. We will try scaling the API endpoint, attempting to simulate an overload of requests. To do this, first, we will check the state we start from: `docker-compose ps`

And then apply the scaling by executing the following command: 
```
docker-compose up -d --scale web_api=2 
docker-compose ps
```
We see that it has been scaled and has set the automatic name of the new containers using the suffix -1 and -2. In the CREATED column, we can see when the second container was created, and in status how long each has been running.

Try the new API endpoint, which in this case is on port 7014.

Now, scale back down to return to the initial state: 
```
docker-compose up -d --scale web_api=1 
docker-compose ps
```

### Uninstalling the Solution and Cleaning Up from our PC

To uninstall the solution, first, let's see what we have: `docker-compose ps`

Remove the docker-compose deployment and verify that it has been removed: 
```
docker-compose down 
docker-compose ps
```

Also, check how they have disappeared in Docker desktop.

Now, we will remove the volumes. First, let's review the volumes we have: `docker volume ls`

And delete them by: `docker volume rm datahack-kubernetes-dotnet-api-mongodb_mongodb-volume`

Check that it is no longer listed because it has been deleted. If we were to bring everything up again, the volume would be recreated and empty.

Finally, delete the images by: 
```
docker rmi [username]/web_api:v1 
docker rmi [username]/web_api:v2
```


## Kubernetes demo
### Solution Design

A basic microservices architecture has been established using Kubernetes, comprising:

- **Frontend (mongo-express)**: for managing the MongoDB database.
- **MongoDB database**: providing data storage.
- **Web API**: utilizing this MongoDB database.

The Kubernetes YAML files include:

**MongoDB**:
- MongoDB Deployment: Deploys a MongoDB instance as the database. It also uses credentials stored in a Secret for initial configuration.
- MongoDB Secret: Stores MongoDB admin credentials in a Secret object for secure use by other parts of the Kubernetes system.
- MongoDB Service: Creates a service to expose MongoDB within the Kubernetes cluster.

**Mongo Express**:
- Mongo Express Deployment: Deploys a Mongo Express instance, a web interface for managing MongoDB. Configured to obtain MongoDB access credentials from a Secret and the database URL from the previously defined ConfigMap.
- Mongo Express Service: Creates a service to expose Mongo Express outside the Kubernetes cluster, using a specific port.

**Web API**:
- Web API Deployment: Deploys a web API utilizing MongoDB as its database. Configured to access the database using access credentials.
- Web API Service: Creates a service to expose the web application within the Kubernetes cluster.

**ConfigMap**: Defines a configuration for the MongoDB database URL. This configuration will be used in the Mongo Express resources to connect to the database.

**PersistentVolume and PersistentVolumeClaim**: Configures persistent storage for MongoDB. Defines a PersistentVolume to store database data and a PersistentVolumeClaim to request that storage.

### Solution Deployment, Running Commands within a Container, and Using the Web Interface

We'll start by configuring the context: `kubectl config use-context minikube`

**Deploying MongoDB:**

We apply the YAMLs:
```
kubectl apply -f namespace.yaml 
kubectl apply -f persistentvolume.yaml 
kubectl apply -f persistentvolumeclaim.yaml 
kubectl apply -f mongo-secret.yaml 
kubectl apply -f mongodb.yaml
```

**Entering the Mongo DB console to insert data:**

Copy the pod name by: `kubectl get all --namespace maialen-namespace`

Enter the console and authenticate: 
```
kubectl exec -it <mongo-pod> --namespace maialen-namespace mongo 
use admin 
db.auth("root", "password") 
use testdb
```
Insert data:
```
db.cars.insert({name: "Audi", price: 52642}) 
db.cars.insert({name: "Mercedes", price: 57127}) 
db.cars.insert({name: "Skoda", price: 9000}) 
db.cars.insert({name: "Volvo", price: 29000}) 
db.cars.insert({name: "Bentley", price: 350000}) 
db.cars.insert({name: "Citroen", price: 21000}) 
db.cars.insert({name: "Hummer", price: 41400}) 
db.cars.insert({name: "Volkswagen", price: 21600}) 
db.cars.find().pretty() 
exit
```

**Deploying the web API:**

For deploying the web API, we need to obtain the IP of the MongoDB service for the API to connect to it. To extract it, we can execute the following command: `kubectl get services --namespace maialen-namespace`
We have to copy the IP, and with this address, we will modify the web-api.yaml to indicate an appropriate mongodb URL in the env variable Mongodb (this part could be automated in future versions).

Save and apply the YAML: 
```
kubectl apply -f web-api.yaml
```

Check that everything is running and/or created correctly: 
```
kubectl get all --namespace maialen-namespace
```

We can also check the log status within the web API pod: 
```
kubectl logs <web-api-pod> --namespace maialen-namespace
```

**Accessing the API web:**

To access the API web, execute the command: 
```
minikube service web-api-service -n maialen-namespace
```

This command makes Minikube open a tunnel from your local machine to the web-api-service that is deployed in the Kubernetes cluster (specifically in the maialen-namespace namespace). This allows accessing the service from your local web browser as if it were running directly on your machine, using the IP address and port associated with the service exposed by Minikube.

**Deploying Mongo-express:**

Open another terminal `cd kubernetes`
Apply the files: 
```
kubectl apply -f mongo-configmap.yaml
kubectl apply -f mongo-express.yaml
```

Check that everything is running: `kubectl get all --namespace maialen-namespace`

To access the Mongo Express web interface, execute: `minikube service mongo-express-service --namespace maialen-namespace`

Like in the previous case, Minikube opens a tunnel from your local machine to the mongo-express-service that is deployed in the Kubernetes cluster. This service is configured with a NodePort service type, which means that the Kubernetes cluster automatically assigns a port on each cluster node (node port) for the service to be externally available.
In this case, the mongo-express-service is configured to listen on a specific port on the cluster nodes (node port, in this case, it is 30000) and redirect traffic through that port.
Therefore, you can access the Mongo Express web interface from your local browser in two different ways:
- Using the IP address of your Minikube node (you can check it by running minikube ip) and port 30000.
- Using the created tunnel and the assigned port through the last command.

We will follow the second option. It opens the web page, and the first time it will ask for a username and password, which are "admin" and "pass". Once entered, we access the web interface.

At this point, we suggest considering using a load balancer instead of a NodePort. A Load Balancer would be preferable when anticipating high traffic. A load balancer distributes the load among multiple instances of a service, which helps avoid congestion and ensures high availability and performance. In a cloud production environment, it would be common to use a load balancer provided by the cloud provider to efficiently and scalably handle external traffic (to be considered in next versions).

Using the web interface, we can try deleting one of the database records. Next, we can update the API to verify that the deletion has been reflected in it as well.

### Database Persistence

To implement data persistence, we utilized the MongoDB Deployment along with a Persistent Volume and Persistent Volume Claim.

**Removing deployment, Service, Pod, Replicaset related to the namespace:**

To verify the proper implementation of persistence, we'll attempt to delete everything related to the namespace and then redeploy everything.

Deletion and verification:
```
kubectl delete all --all -n maialen-namespace 
kubectl get all --namespace maialen-namespace
```

Re-deploying:
Apply the MongoDB Service and Deployment:
```
kubectl apply -f mongodb.yaml
```
Apply the web API Service and Deployment. To do this, as before, first obtain the IP:
```
kubectl get services --namespace maialen-namespace
```
Modify the IP in the yaml and apply the web API Service and Deployment:
```
kubectl apply -f web-api.yaml
```
Finally, also apply the Mongo Express Service and Deployment:
```
kubectl apply -f mongo-express.yaml
```
Check that everything is up and running:
```
kubectl get all --namespace maialen-namespace
```
Access the service:
```
minikube service web-api-service -n maialen-namespace
```
And verify that the data is still there.

### Resource Usage Configuration

Resource usage for the solution is described in the YAMLs of the MongoDB and web API Deployments. To check that the resource usage configuration has been applied to the pods, we can do the following:
```
kubectl get all --namespace maialen-namespace
kubectl describe pod <web-api-deployment-id> -n maialen-namespace
kubectl describe pod <mongo-deployment-id> -n maialen-namespace
```
To view the current resource usage, we first need to enable the metrics server addon. To do this, print the list of addons:
```
minikube addons list 
minikube addons enable metrics-server
```
Wait until it's activated. We can monitor its deployment using this command:
```
kubectl get deployment metrics-server -n kube-system
```
Once deployed and running, verify that it's enabled in the list of addons:
```
minikube addons list
```
Now we can check the metrics:
```
kubectl top pods --namespace maialen-namespace
```

### Scaling the Solution

In this section, we'll try two scaling methods. First, we'll manually modify the number of replicas, and then we'll try configuring automatic scaling.

**Manually modifying the number of replicas:**

To scale manually, execute the following commands:
```
kubectl get all --namespace maialen-namespace
kubectl scale deployment web-api-deployment --replicas=3 -n maialen-namespace
kubectl get all --namespace maialen-namespace
```
From the output, we can see that we've gone from having just one pod of the web API to now having three. We'll scale back down to leave it as it was:
```
kubectl scale deployment web-api-deployment --replicas=1 -n maialen-namespace
kubectl get all --namespace maialen-namespace
```

**Configuring automatic scaling:**

To set up autoscaling, execute this command:
```
kubectl autoscale deployment web-api-deployment --cpu-percent=40 --min=1 --max=10 -n maialen-namespace
```
- When the average CPU usage percentage exceeds 40%, it will autoscale.
- The minimum number of replicas in the deployment is 1, and the maximum is 10.

The following command shows the Horizontal Pod Autoscalers (HPA) in the maialen-namespace namespace:
```
kubectl get hpa -n maialen-namespace
```
Here we can see that it has been configured. We could consider conducting a stress test to evaluate the autoscaling behavior. However, for brevity, this section has not been further extended.

### Uninstalling the Solution and Removing Everything from Your PC

The following set of commands aims to delete all resources related to a deployed environment in Kubernetes. It starts by removing all Horizontal Pod Autoscalers (HPA) specific to the web-api-deployment, then deletes all resources (pods, services, etc.) in the maialen-namespace namespace, followed by the deletion of PVCs and the persistent volume (PV) associated with MongoDB. Finally, it removes secrets and the configuration of the ConfigMap, and deletes the maialen-namespace namespace entirely, thus cleaning up all resources and data related to the environment in Kubernetes.
```
kubectl delete hpa web-api-deployment -n maialen-namespace 
kubectl get all --namespace maialen-namespace 
kubectl delete all --all -n maialen-namespace 
kubectl delete persistentvolumeclaim mongo-data-pvc --namespace maialen-namespace 
kubectl delete pv mongo-data-pv 
kubectl delete secret mongodb-secret -n maialen-namespace 
kubectl delete configmap mongodb-configmap -n maialen-namespace 
kubectl delete namespace maialen-namespace
```

## Improvements for Future Versions
### web-api.yaml File
**Username and Password**

In the web-api.yaml file, as an environment variable "name: Mongodb; value: "mongodb://root:password@10.100.186.15:27017"", we have specified the root username and password for MongoDB. This information is included in the definition of the web API Deployment and is not elegant as it directly exposes the MongoDB database access credentials in the Kubernetes manifest. This poses a security issue since anyone with access to the Kubernetes manifest could view the database access credentials. A more secure and elegant way to handle database access credentials is by using Kubernetes Secrets, as we have done with Mongo Express. This aspect remains pending improvement for future versions.

**MongoDB Service IP Address**

Furthermore, the IP address 10.100.186.15 mentioned in the configuration is not elegant or advisable because the IP address changes with each restart of the MongoDB service. This means that we would need to manually update the configuration each time the service restarts, which can be error-prone and tedious. Instead of directly using IP addresses, it is more elegant and recommended to use hostnames or Kubernetes services. This aspect also remains pending improvement.

### Deployment vs StatefulSet for MongoDB Persistence
In general, both Deployments and StatefulSets can be used to implement persistent databases in Kubernetes. However, there are some important considerations to keep in mind:
- **StatefulSets** are specifically designed for applications that require persistent identity and data storage, such as databases. They provide guarantees of deployment order, stable network identity, and storage persistence, making them ideal for implementing databases in Kubernetes.
- **Deployments** are suitable for applications that do not have persistent states or can handle state loss gracefully. They are more suitable for web applications, APIs, and services that can scale horizontally and where the state is stored externally, such as in a separate database.
In my case, due to time constraints, I used a Deployment for MongoDB instead of a StatefulSet, as the Deployment was the first one I managed to implement. Although it worked to meet the persistence requirements, it would be more advisable to use a StatefulSet for databases in production due to the additional guarantees it provides in terms of data persistence and pod identity stability. This option should be considered for future versions.

### Rolling Update for the Web API Service
As an extension to the Kubernetes section, I would have also liked to explore the implementation of a Rolling Update for the web API service, allowing for a smooth transition from version 1 to version 2 of the application using the images created in the Docker section. This gradual update process would ensure service continuity with minimal disruptions to end-users, which is crucial for maintaining stability and user experience during the deployment of new application versions.