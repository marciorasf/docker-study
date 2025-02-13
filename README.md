# Docker Fundamentals

This repo is a guide to learning the fundamentals of Docker. I wrote it while I was learning to help me cement the knowledge acquired, and of course, to be a guide to help others.

I tried to write only the basics here, as the repo name says, only the fundamentals to start using Docker properly.

## What is Docker

Docker is a software that providade several tools to work with containers with efficiency, both to the development perspective and the execution performance.

### What is a container

A container is an isolated environment, with the configurations and dependencies needed to execute apps properly inside it.

The "isolated" characteristic is essential to the idempotency provided by the use of containers, which means that no matter where the container is executed, the behavior should be the same.

### Container vs Virtual Machine

The main difference between **containers** and **virtual machines**, is that containers virtualize the **operating system**, while virtual machines virtualize the **hardware** to run multiple OS instances.

Consequently, the biggest advantage of using containers is the reduction of the overhead on storage, memory and CPU resources. This can achieved because each container doesn't have to have its own OS.

This advantage can be seen clearly when we compare the container size and the initialization time. Most containers have only megabytes in size and take seconds do start, while VMs have gigabytes and take minutes to initialize.

---

## Why use Docker

- Idempotency - ensures that the app has the same behavior independently of the environment.

- Faster development - the development environment can be set in some minutes on each machine.

- Quick and simple deploy - most cloud computer services accepts deploying container images.

- Sacalability - because the containers are self-contained, they can be replicated to scale apps.

---

## Basic components

### Docker Client

Docker Client serves as an interface to enable the communication between the users and the Docker Engine.

When you enter some command like **docker container ls**, this command is received by the Docker Client so it can be passed to the Docker Daemon.

### Docker Daemon

The Docker Daemon is responsible to listen for Docker Client requests and based on them, manage Docker objects such as images, containers, networks and volumes.

### Docker Registry

A **Docker Registry** stores Docker images. The default registry used with Docker is the Docker Hub, which is a kind of GitHub for Docker images. But you can use private services too, like the AWS ECR or the Google Cloud Container Registry.

---

## How to use Docker

### Requirements

Docker must be installed. The installation guide can be found [here](https://docs.docker.com/engine/install/ubuntu/).

To this specific tutorial is recommended to fork this repo, so you can use the server folder on it to create your own example image.

### Container

A container is a isolated environment used to run apps inside it. The "isolated" characteristic is essential to the idempotency provided by the use of containers, which means that no matter where the container is executed, the behavior should be the same.

#### Run container

You can a container with the following command:

```bash
# docker container run <image>
docker container run hello-world
```

Note that when this command is executed, the first line printed is (if you doesn't have this image on yout computer):

```bash
Unable to find image 'hello-world:latest' locally
```

After the message, the image starts to be downloaded. The download is made from Docker Hub. Next, the container is created, the app is executed and then the process ends.

If you run the same command again, the image will be available locally, so it doesn't have to be downloaded again.

#### Dettached mode

The hello-world container is created, executed and the finishes. But this is not the expected behavior for several other containers. Let's see an example using an nginx container.

```bash
docker container run nginx
```

After creating the container, you can see that the container stay alive (use **ctrl + c**, to stop the process). To not have to keep a terminal locked with this process, you can use the **dettached mode** with the **-d** option.

```bash
docker container run -d nginx
```

After the container creation, the process exits and container keeps running on background.

#### List containers

To see the your containers, use the following command:

```bash
docker container ls
```

If everything is all right until here, this command should list the dettached nginx container.

Note that the hello-world container are not listed, this occurs, because they're stopped. To list all containers use **-a** option.

```bash
docker container ls -a
```

Now you should see the other containers.

#### Execute command on running container

You can execute commands inside containers with:

```bash
docker exec <container_name|container_id> <command>
```

#### Attach to running container

After start a container in dettached mode, it may be necessary to attach to it again. To do it, use the **attach** command:

```bash
docker attach <container_name|container_id>
```

#### Port bind

By default, the containers ports aren't accessible from outside docker. To overcome this problem, you can bind a container port with a host port using the **-p** option:

```bash
# docker container -p <pc_port>:<container_port> nginx
docker container run -p 8080:80 nginx
```

Now, a nginx welcome should be displayed on <http://localhost:8080>

#### Create named container

You also can create a container with a specific name:

```bash
docker run --name mynginx nginx
```

#### Interactive mode

Some containers can run in interactive mode. For example a ubuntu container that can open a terminal. To achieve this, you have to use the **-it** option:

```bash
docker container run -it ubuntu /bin/bash
```

Now you're inside the container and can use commands like if you were on your PC own terminal.

Enter **exit** command to exit container.

#### Stop container

A container runs forever if you don't stop it. The command to stop a running container is:

```bash
docker container stop <container_name|container_id>
```

#### Start stopped containers

Everytime you enter **docker container run** a new container is created. Sometimes you should prefer restart stopped containers, instead of creating new ones. To do it, use :

```bash
docker container start <name|id>
```

Let's see an example:

```bash
# start a nginx container
docker container run -d --name my_nginx nginx:1.19-alpine

# list you running containers. You should see the created nginx container
docker ls

# stop your container
docker container stop nginx:1.19-alpine

# list your running containers and see that the nginx container is not on the list
docker ls

# restart your container
docker container start my_nginx

# list your running containers again and see that the nginx container is running
docker container ls
```

#### Remove container

When using Docker frequently, it's common that your pc ends up as a container cemetary. You can remove these dangling containers.

```bash
docker container rm <container_name|container_id>
```

You can pass more than one container too:

```bash
docker container rm <container_name|container_id> <container_name|container_id>
```

Other option is using **prune** to remove all stopped containers:

```bash
docker container prune
```

### Image

A image is file composed of several layers that works as instructions to create containers. A image is file composed of several layers that works as instructions to create containers.

#### Create image

I will use a simple NodeJS server to explain the image part of Docker. This server is implemented on **server** folder.

To  do the following steps correctly, your terminal must be on the **server** folder.

If you have NodeJS installed, you can start the server using:

```bash
node index.js
```

This message should be displayed:

```bash
Example app listening at http://localhost:3000
```

If you access the link, you can see a "Hello World!". This means that the server is running correctly.

Now we can create an image to this server.

First, we must create a file named Dockerfile.

The content of the Dockerfile is:

```Dockerfile
# Base image
FROM node:14.16.1-alpine 

# The dir that we will work inside the docker
WORKDIR /app

# Copy package.json and yarn.lock
# It's a good practice copy only this files first to use the Docker cache system
# Docker can identify if the commands have some changes
# If it doesn't have changes, it uses the cached layers
# This process can reduce significantly the build time
# As the dependencies do not change frequently, in most cases it will use the cache
COPY package.json .
COPY yarn.lock .

# Install the dependencies
RUN yarn

# Copy the other files
COPY . .

# Define environment variables
ENV PORT=8080

# Expose the port to communicate with the container outside
EXPOSE 8080

# Command that should be executed after the container creation
CMD ["node", "index.js"]
```

There is a problem with this Dockerfile at:

```Dockerfile
COPY . .
```

At this point all files will be copied to the image, even the node_modules. To solve the problem, we should create a **.dockerignore** file. This file has the same format of a .gitignore.

```.dockerignore
node_modules
```

Now we can build the image:

```bash
# docker image build -t <image_name> <path_to_context>
# To this command works correctly your terminal should be in server folder
docker image build -t image_example .   
```

If the Dockerfile is not at the same folder where your terminal is, you can provide a path to it:

```bash
docker image build -t image_example -f ./Dockerfile .
```

A build’s context is the set of files located in the specified PATH or URL.

If you want to see the cache working, you can run the build command again and see the "---> Using cache" messages.

#### List images

Now let's see the created image:

```bash
docker image ls
```

You should see the created image and the others images used until now.

#### Run container with created image

With the image created, let's run a container using the image:

```bash
docker container run --name example -p 8080:8080 image_example
```

You should see the following message:

```bash
Example app listening at http://localhost:8080
```

And if you access the link you can  see the same "Hello World!" message.

#### Push to Docker Hub

As said in the intro, the Docker Hub is the default Docker registry.

Now we will push our image to the Docker Hub.

First you need to register yourself on [Docker Hub](https://hub.docker.com/).

After been already register, you can login in Docker using the terminal:

```bash
# Your credentials will be asked, just enter them
docker login
```

While logged in, you can push your imagem to Docker Hub.

To do this you have to build your image using a name with your namespace on Docker Hub, like marciorasf/example. Also, is a good practice to always tag your images.

Let's rebuild the imagem with the correct name and a tag:

```bash
# docker image build -t <namespace>/<image_name>:<tag> <path_to_context>
docker image build -t marciorasf/image_example:v1 .
```

Now we can push our image to Docker Hub:

```bash
# docker push <namespace>/<image_example>:<tag>
docker push marciorasf/image_example:v1
```

If you don't no your namespace you can find it on the Docker Hub page:

![Docker Hub namespace](./assets/docker_hub_namespace.png)

Another good practice, is to always push a image with the **latest** tag when you push some now version.

To do this, first we need to tag the image with **latest**:

```bash
# docker tag <namespace>/<image_name>:<existent_tag> <namespace>/<image_name>:<new_tag>
docker tag marciorasf/image_example:v1 marciorasf/image_example:latest
```

Then, push it to Docker Hub:

```bash
docker push marciorasf/image_example:latest
```

### Network

When using Docker containers, frequently we have to enable the communication between containers. For this purpose, there are **networks**. As we can infer by the need to expose the ports of the container to the host using **bind**, the default behavior of Docker is having an internal network.

Actually, Docker has a default network that all containers uses if a specific network is not defined.

In this topic I'll show you how to create a network and use it with the containers.

#### Drivers

Before creating networks, we must know that each network needs a **driver**. A driver is like a pluggable config that enables Docker networks to do intede Docker has several network drivers by default that can be used to accomplish the desired network configuration. I won't dive in the several available drivers because it's a more advanced topic than the intend of this tutorial.

For the moment, I believe that you should only know that the default driver is the **bridge**, that are used when standalone containers needs to communicate with each other.

If you want to know more about networks, check the [oficial docs](https://docs.docker.com/network/).

#### Create a network

```bash
docker network create -d bridge network_example
```

Now let's see the created network:

```bash
docker network ls
```

#### Using the network with containers

We'll use a MongoDB and a MongoExpress (SBGB for mongo) container and communicate them using our created network:

```bash
# run the MongoDB container. We have to name the container so the MongoExpress can find it
docker container run -d --net network_example --name my_mongo mongo:4.4

# run the MongoExpress container binding the port, so we can access via browser
# we have to pass the environment variable ME_CONFIG_MONGODB_SERVER using -e 
# so the MongoExpress can find the MongoDB container
docker container run --net network_example -p 8081:8081 mongo-express:0.54
```

Now both the MongoDB and MongoExpress containers are running in the **network_example** network. You should be able to open MongoExpress on http://localhost:8081.

#### List the available networks

You can list your networks using:

```bash
docker network ls
```

#### Remove a network

You can also remove a network using:

```bash
docker network rm <id|name>
```

### Volume

Docker volumes are used for data persistence when using container on Docker. The most clear example of the need of volumes is the persistence that is needed for running databases.

For example: if you're using a MongoDB inside a Docker container, you can save files inside and it will work properly as long as the container keeps running. But, if for some reason the container has to stop, when it starts again the database will be empty.

To understand the volumes on Docker, you need to know that when you start a container, it has a virtual file system which is the place that the container stores the data. What a volume does is basically mount a folder of the host file system into the virtual file system of the Container. After been mounted, everything that stored on that virtual file system folder will be replicated to host folder.

#### Volume types

There are 3 volume types in Docker. Let's take a look on them.

For the demos will use a MongoDB and a MongoExpress containers like we created on the network section.

##### Host Volume

This type of volume is created when you pass both the container directory and the host directory that should be mounted:

```bash
# docker container run -v <host_dir>:<container_dir> <image>
docker container run -v /home/mount/data:/data/db -d --net network_example --name mongo mongo:4.4
```

Now, you can verify that the folder **/home/mount/data** was created. I highly recommend you to run the MongoExpress container and save some data on the MongoDB then restarts the container and see that the data is really there after the restart.

You can remember easily of which volume type is the **host volume**, by remembering that this is the volume that you need to specify the host directory. 

On the next types I keep my recommendation to you do tests using the MongoExpress and restarting the MongoDB container. So I won't keep repeating myself.

##### Anonymous Volume

This type of volume is created when you just specify the container directory. When you do that you tell Docker that you want to persist that specific folder but don't care where on the host it should be. So the Docker manage this for you.

```bash
docker container run -v /data/db -d --net network_example --name mongo mongo:4.4
```

You can remember which volume type is the **anonymous volume**, by remembering that is the case which you don't specify a host directory so Docker creates automatically one and you know nothing about the directory except that was created by Docker and the container data is persisted inside it. 

##### Named Volume

Lastly, the named volume is a "improvement" of the anonymous volume. When using a named volume, you can reference the volume to be used by its name. 

This is the type of volume you should use on production.

```bash
# when using named volumes, you need to create them first
docker volume create volume_example

# now you can use it
docker container run -v volume_example:/data/db -d --net network_example --name mongo mongo:4.4
```

#### List the available volumes

You can list your volumes using:

```bash
docker volume ls
```

#### Remove a volume

You can also remove a volume using:

```bash
docker volume rm <name>
```


## Docker Compose

### Requirements

You have to have the DockerCompose installed. You can find the installation guide [here](https://docs.docker.com/compose/install/).

DockerCompose is a wonderful tool to ochestrate Docker containers. Using DockerCompose you can start several containers based on a yaml file (usually called docker-compose.yaml). This enables you to starts your containers in a much faster way, including all configurations we saw on whis guide.


### Running containers

Let's take a look on an example of a docker-compose.yaml where I added a MongoDB, MongoExpress and Nginx containers.

```yaml
version: "3"

services:
  mongodb:
    image: mongo:4.4
    container_name: my_mongo
    ports:
      - 27017:27017
    volumes:
      - my_mongo_data:/data/db
    networks:
      - my_mongo_network

  mongo-express:
    image: mongo-express:0.54
    container_name: my_mongo_express
    ports:
      - 8081:8081
    networks:
      - my_mongo_network
    environment:
      ME_CONFIG_MONGODB_SERVER: my_mongo
    
  nginx:
    image: nginx:1.19-alpine
    container_name: my_nginx
    ports:
      - 8080:80
      
volumes:
  my_mongo_data:

networks:
  my_mongo_network:
    driver: bridge
```

The network configuration is not needed when using DockerCompose, because it creates a network automatically to enable the containers declared inside the .yaml to communicate with each others.

To start the containers, just enter:

```bash
docker-compose up
```

Now you should see the all containers' logs mixed.

#### Stopping the containers

You can stop the containers typing **ctrl+c**. When you type **ctrl+c** the DockerCompose will send signals to the containers so they can stop properly. You also can type it again so it'll force the containers to stop immediately.

#### Initialization order problem

It's important to note that when you start the containers, they start simutaneously. This means that we could have some problems with MongoExpress because it can starts much before MongoDB, so it wouldn't connect correctly.

On the docker-compose.yaml you can configure the order of service startup using **depends_on**. Let's see how it's used:

```yaml
version: "3"

services:
  mongodb:
    image: mongo:4.4
    container_name: my_mongo
    ports:
      - 27017:27017
    volumes:
      - my_mongo_data:/data/db
    networks:
      - my_mongo_network

  mongo-express:
    image: mongo-express:0.54
    container_name: my_mongo_express
    ports:
      - 8081:8081
    networks:
      - my_mongo_network
    environment:
      ME_CONFIG_MONGODB_SERVER: my_mongo
    depends_on:
      - mongodb
    
  nginx:
    image: nginx:1.19-alpine
    container_name: my_nginx
    ports:
      - 8080:80
      
volumes:
  my_mongo_data:

networks:
  my_mongo_network:
    driver: bridge
```

With this change, the DockerExpress container will only start when the MongoDB container is already started.

You can find this docker-compose.yaml file on the root of this repo.


### Removing created containers, networks, images and volumes

Instead of removing the containers, images, networks and volumes one by one after using the **up** command, you can remove them with **down** command:

```bash
docker-compose down
```

This command by default will stop and remove the containers and networks created. See the [official command docs](https://docs.docker.com/compose/reference/down/) for all the options available.

## Miscellaneous

I've put in this section some additional information and tips that may help you using Docker.

### Good references

- [TechWorld with Nana](https://www.youtube.com/watch?v=3c-iBn73dDE)
- [Amigoscode](https://www.youtube.com/watch?v=p28piYY_wv8)
- [Docker](https://docs.docker.com/)

### Docker command default skeleton

Looking back our tutorial, we can see that the Docker commands assume a kind of a standard skeleton. Understanding the skeleton, you can infer a lot of commands without having to consult the docs. The skeleton is the following:

```bash
docker <object_type> <command>
```

Some examples:

```bash
docker container list
docker image list
docker volume list
docker image rm <id|name>
docker container rm <id|name>
```

### prune

The **prune** command is a convenient way to clean your docker objects

```bash
# docker <object_type> prune
# object_type can be: container, image, network, volume or system
docker system prune
```

### ps

The **ps** command is the same as **docker container ls**. I prefer using **docker container ls** because it's more explicit.

```bash
docker ps
```

### logs

A useful command to see the logs of a container running on dettached mode is:

```bash
docker container logs <id|name>
```

### Pass a file to docker-compose

When you have a file named **docker-compose.yaml**, DockerCompose automatically recognizes it. But you can specify a file using **-f** option. This is very useful when you have different containers for development and testing for example.

```bash
docker-compose -f container.yaml up
```

---

<br/>

<h2 style="text-align: center;"> That's all folks!</h2>

<h3 style="text-align: center;"> Check also my <a href="https://github.com/marciorasf/kubernetes-fundamentals">Kubernetes Fundamentals</a>
