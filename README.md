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
