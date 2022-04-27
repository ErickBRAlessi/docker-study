# Docker Studies
My own studies of Docker

## Goals
* Isolation 
* Resource management
* Version Control

## How do they work

It basically generates a isolated process inside the original OS. The containers can provide isolation in many levels through namespaces.


The isolation levels are divided in:
* PID - Provides isolation of the process running inside the container
* NET - Provides isolation of web interfaces
* IPC - Provides isolation of communication between process and shared memory 
* MNT - Provides isolation of the file system / build point
* UTS - Provides kernel isolation. Handling the container like It were another host

The Cgroups will provide the resource management in terms of memory and CPU.

## Installing Docker on Linux

Follow the documentation:
https://docs.docker.com/engine/install/ubuntu/

To not use sudo before every command you can place your user in a group called docker

``
sudo usermod -aG docker $USER
``

It will be necessary to restart the system.

For testing the following command can be run:

``
docker run hello-world
``

## Docker Hub

Docker hub is a docker images repository. You can find images to be executed there.

The following command will download the image to be run latter.

`` docker pull IMAGE ``

The docker run command will look for the image locally, if it doesn't find, it will try download it from docker hub, then the image hash will be validated and finally the image will be run.

`` docker run IMAGE [COMMAND] `` sleep 1d can be used to keep a container on, since it needs at least a process running to not be closed.

You can see the running images with:

`` docker ps `` 

or both the running images and the already closed

`` docker ps -a``

or it brings also the virtual container sizes

`` docker ps -s``




### Other Container commands

| command | result |
| :---: | :---: |
| `` docker [TOPIC] COMMAND  `` | It brings all topic related command. Ex: image, volume  |
| `` docker run -it [IMAGE] bash  `` | Runs directly in the terminal  |
| `` docker run -d [IMAGE]  `` | Runs in background and prints container ID  |
| `` docker [-t=0] stop [ID/NAME] `` | Stops a container |
| `` docker stop $(docker container ls -q) `` | Stops all containers |
| `` docker pause [ID/NAME] `` | Pauses a container (do not kill the internal processes) |
| `` docker rm [ID/NAME] `` | Removes a docker, it cannot be restarted |
| `` docker rm $(docker container ls -aq) `` | Removes all containers, including the stopped ones |
| `` docker start [ID/NAME] `` | Restarts a container |
| `` docker exec -it [ID/NAME] bash `` | Opens the container on the bash  |
| `` docker port [ID/NAME] `` | Shows the ports mapping  |
| `` docker images `` | Shows the available images  |
| `` docker inspect [Image Id] `` | Shows the image details  |
| `` docker history [Image Id] `` | Shows the image layers  |



## Mapping ports

The ports inside the container are not automatically configured. For example, when you run 
`` docker run -d dockersamples/static-site `` , It will run a container with a website running on port 80. However, you are not going to be able to access it from localhost:80.

To access the site will need to map the ports. Instead of the last command, you can run add the flag -P 

``docker run -d -P dockersamples/static-site ``

You can see now that the internal port (port 80) was mapped to another host port and it can be accessed from terminal.

This result can be achieved in a more elegant and well defined way. 

``docker run -d -p 8080:80 dockersamples/static-site ``

The host port 8080 will return the container port 80. 

## Images
### What are they?
Images are a group of layers. Those layers are independent from each other and each one has its own ID. The images are read only, however, when it's run, the docker add an additional layer of reading/writing.
So the basic process is:

 **dockerfile** *-build->* **image** *-run->* **container**

### Making a new Image
1. Make a new file called: Dockerfile
2. In this file start with the tag **FROM** to define the base (a base can be find on dockerhub). *Ex: FROM node:14*
3. Define a default work folder with the tag **WORKDIR**. *Ex: WORKDIR ./app-node*
4. **ARG** defines a env variable. (Build time)
5. **ENV** defines a env variable (Run time)
6. **EXPOSE** says the internal port of the docker 
7. To copy the host content to this image, use the tag **COPY**. *Ex: COPY . /app-node*
8. To exec a command when the container is being created use the tag **RUN**. *Ex: RUN npm install*
9. **ENTRYPOINT** will run a command after the container is already created. *Ex: ENTRYPOINT npm start*
10. Save the file
11. Use the command ``docker build -t nome/app-nome:1.0 .`` to build a image with nome/app-nome name version 1 in the actual directory.

References: https://docs.docker.com/engine/reference/builder/


### Exporting a image to DockerHub

1. Create a account on DockerHub
2. Login in using ``docker login -u [USER]``
3. Use the command ``docker push [USER/image:version]``

Note that the user should be the same as the image. 


### Images Commands


| command | result |
| :---: | :---: |
| `` docker images `` | Shows the available images  |
| `` docker inspect [Image Id] `` | Shows the image details  |
| `` docker history [Image Id] `` | Shows the image layers  |
| `` docker tag [Image] [New Image]`` | Creates a  copy of the image with a new repository name  |
| `` docker rmi $(docker image ls -aq) --force `` | Removes all images |


## Handling Data

Docker provides three ways to persist data: 
1. Volumes: persist data on a persistent layer.
2. Bind mounts: persists data on a persistent layer based on host folder structure.
3. TMPFS mounts: creates a temporary storage

### Volumes

This is most recommended way to persist data. Because it creates a area inside the host, but the container is the manager. You can run your container linking this volume to the container-folder, if the volume does`t exist, it will be created.

`` docker run –it --mount source=[VolumeName],target=/container-folder ubuntu bash ``

 or (this way you need to have you volume created previously)

`` docker run -it -v [VolumeName]:/container-folder ubuntu bash ``

The volumes files inside the host are going to be in /var/lib/docker/volumes/[VolumeName]/_data

| command | result |
| :---: | :---: |
| `` docker volume ls `` | Shows the available volumes  |
| `` docker volume create [Volume Name] `` | Create a volume with the given name  |
| `` docker volume rm [Volume Name] `` | Removes one or more volumes with the given name  |


### Bind Mount
It allows to use a local folder to link with a folder inside the container. The following command will run a ubuntu container linking the ./host-folder with the ./container-folder.

`` docker run -it -v /host-folder:/container-folder ubuntu bash ``

When you start it, it will create the /container-folder inside the ubuntu and it can be accessed from the host normally. Although, the recommended way to do this is using the *--mount*.

`` docker run –it --mount type=bind,source=/host-folder,target=/container-folder ubuntu bash ``

### TMPFS mounts
It just works in Linux. It creates a in memory storage. It will not write on the layer of read-write. To create it:

`` docker run -it --tmpfs=/app ubuntu bash `` 

or 

`` docker run –it --mount type=tmpfs,destination=/container-folder ubuntu bash ``


## Networks
Every container will run with a configured network. If it is not set previously, the docker will automatically configure it on a default bridge network for you. You can check the container network id and its ip with the inspect command or 
`` docker network ls `` to list all the available networks. You should be able to communicate all these container through tcp/ip. 

### Bridge
The bridge network is used to communicate containers running on the same host. You may create your own user-defined bridge network and be able to communicate the containers through a automatic DNS resolution, it means, through the container's name. To create the network use the command:

`` docker network create --driver bridge [Network Name] ``

then when you are about to create the container, you can set this network and the selected container name with:

`` docker run -d --name [Container Name] --network [Network Name] ubuntu bash ``

Now you should be able to communicate between the containers using, for example: ping [Image Name].


### None and Host

If you create a network using the driver none, it means that the container will be isolated in terms of network. 

`` docker run -d --network none ubuntu bash ``

If you create a network using a driver using host, it will use the host network. It may be useful if you don't want to use the docker port mapping. 

`` docker run -d --network host ubuntu bash ``


## Docker Compose

The main goal of the docker compose is to solve the "how to run multiple containers at same time in a coordinated way", so it wouldn't be necessary to execute each run command individually. It allows to create an .yml file used to create a "recipe" to run many containers at the same time.

If it is not installed, you should follow the documentation: 
https://docs.docker.com/compose/install/


To create the docker-compose.yml file.
1. First define the **version**. Ex: version: "3.9"
2. Create the **services** that it will run. Ex: services: mongodb: image: mongo:4.4.6 container_name: meu-mongo networks: - compose-bridge ports:3000:8080
3. Create the **networks** that doesn't exists. Ex: networks: compose-bridge: driver: bridge
4. Write down how a container **depends_on** another one. So the docker-compose will wait until the dependencies are created to run the depended container. Ex: depends_on: - mongodb
Now you can save the file and run the command in the folder where it's saved.

`` docker-compose up -d ``

other docker-compose commands:

`` docker-compose ps ``
`` docker-compose down ``

for more info about compose file:
https://docs.docker.com/compose/compose-file/

