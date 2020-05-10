---
published: true
---
Researching for my internship, I was tasked to look into the auto deployment of container service, Kubenetes with google cloud in mind. Frankly, I am not sure about what this service offers even thou I have heard about it's use from my dad before. Looking into it, I found some cool 

# Starting up
> [Scotch Tutorials](https://scotch.io/tutorials/google-cloud-platform-i-deploy-a-docker-app-to-google-container-engine-with-kubernetes "Reference")

The main location: https://kubernetes.io/

Kubentes allow us to deploy docker containers quicker and predictably. 
- Scaling
- New features rolled out quickly
- Easy management (Roll out and rollback)
- Horizontal scalling (Scale depending on cpu)
- Provided statistics (Healthcheck)

## Need to know
- Docker
- Python
- Flask (For intern)

## Install
- Docker
- [glcoud](https://cloud.google.com/sdk/docs/#mac)
- Kubectl: `gcloud components install kubectl`

## Project creation
[Creating first project](https://cloud.google.com/sdk/docs/#mac)

# Clusters
- A group of google engine instances running in kubenetes.
- Work as a single unit

> We can connect to the cluster using the `kubectl proxy` to open a dashboard interface

# Docker image

      #Create our image from Node 6.9-alpine
	  FROM node:6.9-alpine

		MAINTAINER John Kariuki 	<johnkariukin@gmail.com>

		#Create a new directory to run our app.
		RUN mkdir -p /usr/src/app

		#Set the new directory as our working 	directory
		WORKDIR /usr/src/app

		#Copy all the content to the working directory
		COPY . /usr/src/app

		#install node packages to node_modules
		RUN npm install

		#Our app runs on port 8000. Expose it!
		EXPOSE 8000

		#Run the application.
		CMD npm start

- Image has to be in `gcr.io/{$project_id}/{image}:{tag}` format

# Pods
A kube pod is a group of containers that are tied together for admin purpose. 

We will use deployment.yml to config our kube.

Sample `deployment.yml`

		apiVersion: extensions/v1beta1
		kind: Deployment
		metadata:
  			name: scotch-dep
  			labels:
    			#Project ID
    			app: scotch-155622
		spec:
  			#Run two instances of our application
  			replicas: 2
  			template:
    			metadata:
      				labels:
       					 app: scotch-155622
    			spec:
      				#Container details
          containers:
            - name: node-app
              image: gcr.io/scotch-155622/node-app:0.0.1
              imagePullPolicy: Always
              #Ports to expose
              ports:
              - containerPort: 8000

After writing, we can run the `kubectl create -f deployment.yml` to create the pod

# Exposing deployments

`kubectl expose deployment {service_name} --type="LoadBalancer"`

- This allow us to expose our pods for connection to the outside world

We can also create a `service.yml` file to auto create.

Sample `service.yml`:

      kind: Service
      apiVersion: v1
      metadata:
        #Service name
        name: node-app-svc
      spec:
        selector:
          app: scotch-155622
        ports:
          - protocol: TCP
            port: 8000
            targetPort: 8000
        type: LoadBalancer

Running the `kubectl create -f service.yml` command will create the service. We will be able to externally access the ip address in the `kubectl get services` command


# Scaling up/down

`kubectl scale deployment {deployment_name} --replicas=n`

- Scaling depends on the traffic

## Autoscale
`kubectl autoscale deployment nginx-deployment --min=5 --max=10 --cpu-percent=75`

- Set the min and max num of pods base on cpu utilization from existing pods

# Thoughts and feedback

- Docker and kube are very closely tied
- Kube allow us to ensure that our products have minmum downtime regardless of whether we decided to change/update the product
- Scaling is easy
