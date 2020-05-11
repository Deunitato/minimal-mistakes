---
published: true
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

## Inference Graph

Keycomponents of a deployment yaml
- Predictors: Each defines a graph and its set of deploymnets. Multiple predictors is useful when splitting traffic between a main graph and a canary

- componentSpecs: A kubernetes PodTemplateSpec, seldon will build into a kubernetes deployment

	- Images from graph
    - Requirements
- Graph: Specifies how components are joined together.


Complex Example:

```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-model
spec:
  name: test-deployment
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: step_one
          image: seldonio/step_one:1.0
        - name: step_two
          image: seldonio/step_two:1.0
        - name: step_three
          image: seldonio/step_three:1.0
    graph:
      name: step_one
      endpoint:
        type: REST
      type: MODEL
      children:
          name: step_two
          endpoint:
            type: REST
          type: MODEL
          children:
              name: step_three
              endpoint:
                type: REST
              type: MODEL
              children: []
    name: example
    replicas: 1
```

> Note that the number of containers corresponds to the number of graphs
>
> Each graph is a child of a parent

## Deployment
We can deploy it to kubernetes with `kubectl`

Deploying a `my_ml_deployment.yaml` file:

`kubectl apply -f my_ml_deployment.yaml`


We can also check the status by running:
`kubectl get sdep -o jsonpath='{.items[].status}`

## Testing Model Endpoints

Ways to test:
- Run model direct on python client
- Run model as Docker Container
	- Can be for Language wrapper but not prepackaged inference servers
- Run SeldonDeployment in Kubernetes Dev Client such as KIND
	- CLI tools
    - Python client
    - Documentation UI
    
### Python client
Only can be use for python language wrapped models

Create a file call `myModel.py`:
```
class MyModel:
    def __init__(self):
        pass

    def predict(*args, **kwargs):
        return ["hello, "world"]
```

We can run by using the microservice CLI that is provided by the [python module](https://docs.seldon.io/projects/seldon-core/en/latest/python/python_module.html)

`seldon-core-microservice MyModel REST --service-type MODEL`

### Docker container

`docker run --rm --name mymodel -p 5000:5000 mymodel:0.1`

Sending request on curl:
```
> curl -X POST \
>     -H 'Content-Type: application/json' \
>     -d '{"data": { "ndarray": [[1,2,3,4]]}}' \
>         http://localhost:5000/api/v1.0/predictions

{"data":{"names":[],"ndarray":["hello","world"]},"meta":{}}
```

> Curl is a tool to transfer data from or to a server using one of the protocols e.g HTTP
>
> The command is designed to work without user interaction

### Ambassador

- `http://<ambassadorEndpoint>/seldon/<namespace>/<deploymentName>/api/v1.0/predictions`

	- Location: `<ambassadorEndPoint>`
	- Seldon deployment name: `<deploymentName>`
	- namespace: `<namespace>

### Client Implmentation

- We can use curl

`curl -v 0.0.0.0:8003/seldon/mymodel/api/v1.0/predictions -d '{"data":{"names":["a","b"],"tensor":{"shape":[2,2],"values":[0,0,1,1]}}}' -H "Content-Type: application/json"
`

- This assumes a seldonDeployment `mymodel` with Ambassador exposed on 0.0.0.0:8003

> Ambassador is a open source envoy that provides exposure, security and manages traffic to our kubernetes microservice. [Read More](https://www.getambassador.io/docs/latest/about/why-ambassador/)

## Python Seldon Core Module

Typical:
`pip install seldon-core`

Google:
`pip install seldon-core[gcs]`

All dependency:
`pip install seldon-core[all]`

Methods provided:
- `predict`
- `transform-input`
- `transform-output`
- `route`
- `aggregate`
