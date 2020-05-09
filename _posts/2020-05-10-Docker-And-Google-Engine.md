---
published: false
---
# Introduction 

Docker is a containerisation technology where the containers allow us to wrap up any code that we write. 
The difference between this an virtual machine is that the hyperviser requires a fake hardware (which takes up alot of space)

Containers, share the kernal of the host OS.

![docker_1.PNG]({{site.baseurl}}/_posts/docker_1.PNG)

Layers makes the building containers fast. Sharing containers becomes faster because we only need to download the changed layers

> E.g: Changing containers is fast as long as we have the base infomation (we only need to download the changes)

## Why Containers
- Standardised environment

## Docker in action

`docker build -t webinar-flask:v1`

- tagged: webinar-flask:v111
- This builds the container
- We can look at the dockerfile to see what it does

Docker has a registry with the base images. The different repositories were made by the community that push out their own containers

Installing the file:
- We can an create different layers
- `RUN apt-get update -y`
> This will update

- We can use pip to install the dependency (E.g Flask)

`docker run -d -p 5000:5000 webinar-flask:v1`

- d means deamonized
- p is to bind the port of 5000 to the host 5000

`docker ps`
- Show the running container 

> Killing the container using `docker kill <hash>` would remove the container. Reloading the website will close.

- The container allow us to kill/start/stop anytime we want

`docker stop $(docker ps -a -q)`

> Stops the containers

## Multiple containers

- In the **docker-compose.yaml** file, we can specify the number of containers we want
- We can specify the service (E.g redis)

> It will auto fetch if we do not have the service

## Kubernetes
- We can use kubenetes to update the containers
- Set the cluster size 
- A pod = a group of more than one containers

Things needed

1. Replication controller

Help auto create pod for users. 

- **rc-file.yaml**
- All names and version must be matching
- We can specify  the continaers
- Show the image
- Expose the port
> This is similiar to what happen in the docker-compose file

`kubectl get rc`
- This show the number of desired and the current we have

`kubectrl get pods`
- hShow the pods running

`kubectrl get services`
- Show the services

`kubectl describe rc`
- Gives details about the replication controller




2. Service to expose to the world

`kubectrl create -f service-file.yaml`
- Creates a service file
- This are the config files 

- port - is the port that is exposed to the world

## Deploying a new version
`kubectl get rc`
- Show the name iof the rc

`kubectl rolling-update <name>-rc --filename rc-filev9.yaml`
- this will create a new replica
- Ensure that the version is in the pod (template)
- These are already prestored in the container registry

- This will cause the replication controller to scale up slowly (it will take a while)

> There are different ways from updating a single and multiple containers


## Deploying a container in the container registry

> Lets say we make a change already

`docker build -t grc.io/<Nameofyourgoogleproj>/<title>:v11`
- This will allow docker to know where to map into
- This will build the container with the tag v11

`gcloud docker push grc.io/<Nameofyourgoogleproj>/<title>:v11`
- This will push the container into the registry
- We need the credential for our project
- It only push the code that has change instead of sending the layer that already exists.

> We do not have the file needed to deploy this we need to copy the rc file (v11)

- Change everything to v11 
- Ensure the container is the same and the tags are change to vl

`kubectl rolling-update <Nameofthectrl> --filename rc-fileV11.yaml`

- This might take a while for it to load before it fully deploy (Load balancing)

`kubectrl proxy`
- Gives a weblink for user interface
- Allow us to see the stats
> Rmb to use /ui


# Findings/Reflection

- Kubentes allow us to update our application while not destroying users experience (Reduce downtime)
- Pods are clusters of containers
- Each container is a code/application
- Layers can be built on top of each other
- imagelayers.io: show us what each of the layers are doing


Reference: https://www.youtube.com/watch?v=WAPXaDpkytw