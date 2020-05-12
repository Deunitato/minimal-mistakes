---
published: true
---
Steps required to upload a docker image into kubernetes.

### Requirements
- Docker installed
- Gcloud installed
- Kubectl installed

# Creating a docker image using dockerfile and pushing it to container registry

1. Navigate to location of the docker file

Required files
- code
- requirements.txt
- dockerfile

2. Run the build

`docker build --tag gcr.io/science-experiments-divya/times2:preprocess-0.1 .`

> Note that it must be similiar as specified in the yaml file
>
> image: gcr.io/science-experiments-divya/times2:preprocess-0.1

In this case, the name of the python code/class is `times2` and has the tag `preprocess-0.1`

- This will build the docker image

3. Authenticate (Only need to do this once)
Skip this if its done before

`gcloud auth configure-docker`

4.Push the docker image

`docker push gcr.io/science-experiments-divya/times2:preprocess-0.1`

- This will push the image into the container registry under kubernetes

- Can check by going google cloud platform > Container Registry

5. Deploy the containers using yaml file

`kubectl apply -f model_deployment.yaml`

- Can check using `kubectl get services` to see services

or
- by going under google cloud > kubernetes engine > workload

- Can check pods using `kubectl get pods`



During my deployment, my mentor has already installed the seldon core for me and created a manager pod, thus my yaml file was able to work.

[Seldon core github repo](https://github.com/SeldonIO/seldon-core)

## Testing the deployment

