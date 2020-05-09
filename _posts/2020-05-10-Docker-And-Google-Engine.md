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
