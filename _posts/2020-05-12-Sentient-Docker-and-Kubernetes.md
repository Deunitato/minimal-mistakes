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

Basically follows the workflow:

`docker push [HOSTNAME]/[PROJECT-ID]/[IMAGE]`

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

1. We can check the ambassodor api 

`kubectl get service ambassador`

- Ambassador is the gateway to our deployments

![kube_1.PNG]({{site.baseurl}}/img/kube_1.PNG)

2. Find the end point of our ambassador

Google cloud platform > Kubernetes Engine > Ingress and Service

![kube_2.PNG]({{site.baseurl}}/img/kube_2.PNG)

We are trying to connect to  http://35.240.217.69:80

3. Look for the prefix url

GCP > Kubernetes Engine > Ingress and Service > (Name of **pod**) > yaml

Look for the prefix field under the yaml file:

e.g
`prefix: /seldon/default/seldon-model-preprocess/`

4. Connect using curl in bash

Format: `curl -g <ipaddress>/<prefix>/api.v0.1/predictions`


`curl -g http://35.240.217.69:80/seldon/default/seldon-model-preprocess/api/v0.1/predictions -H "Content-Type: application/json" --data @deployment_model/test_data.json`


# Changing the image
Lets say we have updated the code and we want to deploy it back. We need to 
- Rebuilt the image with new tag (We can leave the tag the same and 
- Repush the image
- Delete previous deployment yaml file
`kubectl delete -f deployment.yaml`
- Change yaml file (If tag changes)
- Redeploy
`kubectl apply -f model_deployment.yaml`


# Checking Logs/Debugging

The log file is the best place to find out if we made a mistake in the code.

Navigate to this:

GCP > Kubernetes Engine > Workloads > (Name of **pod**) > Managed Pods > Container logs

Manage pod:
![kube_4.PNG]({{site.baseurl}}/img/kube_4.PNG)


Container Logging:

![kube_3.PNG]({{site.baseurl}}/img/kube_3.PNG)

- Look under traceback, it shows the error with the code

e.g:
Under the traceback in the log we can see the following error
```
Traceback (most recent call last):
  File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 2447, in wsgi_app
    response = self.full_dispatch_request()
  File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request
    rv = self.handle_user_exception(e)
  File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function
    return cors_after_request(app.make_response(f(*args, **kwargs)))
  File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception
    reraise(exc_type, exc_value, tb)
  File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise
    raise value
  File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request
    rv = self.dispatch_request()
  File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request
    return self.view_functions[rule.endpoint](**req.view_args)
  File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 78, in TransformInput
    user_model, requestJson, seldon_metrics
  File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 218, in transform_input
    user_model, features, class_names, meta=meta
  File "/usr/local/lib/python3.7/site-packages/seldon_core/user_model.py", line 245, in client_transform_input
    return user_model.transform_input(features, feature_names)
  File "/app/times2.py", line 13, in transform_input
    return X * 2
TypeError: unsupported operand type(s) for *: 'dict' and 'int'
```
> In this example, I have tried to multiply a dictionary with an int hence the type error in line 13

# Sample Json Input data:

1. jsonData:
`{"jsonData":{"X":4}}`

Note: Using this, functions needs to return a dictionary

2. Normal:
{"data": {"names": [],"ndarray": [[1]]}}

# Good references
[Kubernetes cheat sheet](https://kubernetes.io/docs/reference/kubectl/cheatsheet/)