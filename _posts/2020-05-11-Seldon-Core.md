---
published: false
---
While preparing for my internship, I am required to read up more about this new platform known as Seldon.

Seldon is an open source platform for us to deploy our ml models on kubernetes. 

## How it works
- Converts ML models/Language wrapper into production (REST/GRPC) Microservices

> Microservices is a software engineering technique and a architechture style which is structured as a collection of isolated services
> 
> This makes it easy for deployment and identification of a point of failure.

_To be able to understand how seldon works, part of the job is to understand the workings of AI and ML models. Unfortunately, this is something that I have no experience on so I guess I will try my best to understand it. As part of my journey with this internship_

## Machine Learning (ML Models)

A model is a representation of reality. We use data sets to model our reality. 

The choice of model is based on algorthims. 

# Deep Dive 

## Seldon Core operator chart configuration

- Seldon makes use of `helm` to manage the kubenetes clusteres.
- We can download seldon using the `helm` command

## Prepackaged Model Servers

Following ML model, seldon provide some prepacked servers that allow us to deploy trained models.
- SKLearn Server
- XGBoost Server 
- Tensorflow Serving 
- MLflow Server

These servers allow us to deploy algorthims and experiements while keeping the same server architecture and API (Tensor

Model servers simplify deployment of machine learning in the same way app servers simplify the task of delivering a web app or API to end users.

The model should be saved in a local file store (E.g Google bucket)

Sample manifest with sklearn server:

```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: sklearn
spec:
  name: iris
  predictors:
    - componentSpecs:
    - spec:
        containers:
        - name: classifier
          resources:
            requests:
              memory: 50Mi
    - graph:
        children: []
        implementation: SKLEARN_SERVER
        modelUri: gs://seldon-models/sklearn/iris
        name: classifier
      name: default
      replicas: 1
```
Terms:
- `modelUri`: Specifies the bucket with the model
- `Container`: Ensure that same name to podSpecs

The image name and other details will be added when this is deployed automatically.

> My guess is that the 'graph' is referring to the model for ML


### Setting up seldon
1. Understand the Environment variables
- This differs for different cloud 
2.Create a secret containing the environment 
variables

We can also do so by command:

Sample command for AWS
```
kubectl create secret generic seldon-init-container-secret \
    --from-literal=AWS_ENDPOINT_URL='XXXX' \
    --from-literal=AWS_ACCESS_KEY_ID='XXXX' \
    --from-literal=AWS_SECRET_ACCESS_KEY='XXXX' \
    --from-literal=USE_SSL=false
```

3. SeldonDeployment must have access to the secret

- Following the example above, the name of the secret is `seldon-init-container-secret`

> Note that for google cloud, the file in the secret needs to be called based on the `kubectl get cm -n seldon-system seldon-config -o yaml` command.

- Create a service account to reference the secret (Google cloud)

	- This is because google credentials requried a file to be set up
    - Creating the service account and attaching a secret is similiar to [kfserving](https://github.com/kubeflow/kfserving/tree/master/docs/samples/s3)

e.g
```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: sklearn
spec:
  name: iris
  predictors:
  - graph:
      children: []
      implementation: SKLEARN_SERVER
      modelUri: gs://seldon-models/sklearn/iris
      serviceAccountName: user-gcp-sa
      name: classifier
    name: default
    replicas: 1
```
In this case, the `serviceAccountName` is set to be `user-gcp-sa` which is the same level as `modelUri`

Service account:

```
apiVersion: v1
kind: ServiceAccount
metadata:
  name: user-gcp-sa
secrets:
  - name: user-gcp-sa
```

## Language Core Wrappers (Seldon)
- All prepackaged model servers are built using language wrappers
- We can build reusable inference server if needed


