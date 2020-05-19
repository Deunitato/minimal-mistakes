---
published: true
---

# Installation

1. Ensure that you have the repo inside the cloud

Install the repo:
```
SELDON_VERSION=v1.1.0 SELDON_URL=https://github.com/SeldonIO/seldon-core.git

git clone $SELDON_URL --branch $SELDON_VERSION
```

2. Run the installation command:

- Ensure you are in the right directory (Helm-charts)

`helm install --name seldon-core-analytics ./seldon-core-analytics --repo https://stora
ge.googleapis.com/seldon-charts --namespace seldon-system`


3. Start the interface on local host:

Grafana:
`kubectl port-forward svc/seldon-core-analytics-grafana 3000:80 -n seldon-system`

Prometheus:
`kubectl port-forward svc/seldon-core-analytics-prometheus-seldon 3001:80 -n seldon-system`


4. Open browser and run `localhost:3000`

### Errors:

1. Need 1 argument: Chart Name
- Possibly missing `--name`
Link: https://stackoverflow.com/questions/60490347/helm-install-error-this-command-needs-1-argument-chart-name

2. Missing tiller

> Note: Check if you have run `helm init` first before running the next commands

- Missing service account tiller

```
kubectl create serviceaccount --namespace kube-system tiller

kubectl patch deploy --namespace kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'
```

Check if tiller is working: helm version

3. Error: release seldon-core-analytics failed: podsecuritypolicies.policy "seldon-core-analytics-grafana" already exists

- Remove the psp (Pod security policy)

```
kubectl get psp -n seldon-system

kubectl delete psp seldon-core-analytics-grafana-test -n seldon-system

helm del --purge seldon-core-analytics
```
- Delete all psp seen in the `get`
- Rerun the command again

4. Reoccuring secrets
- Secrets keep regenerating when attempting to delete

> To check secrets, in google cloud go to Kubernetes engine > Configuration

Fix: Delete service account

5. Templating init (When running grafana)

- Encounter a bad gateway after running localhost
- This might be the problem to do with your connection from grafana to the prometheus
- Perform the following checks
	- Datasource > URL (Ensure it is the same name as your deployment)
    - Namespace is the same (Seldon-system) 
    - Naming convention is correct (Ensure that your deployment name starts with seldon-core-analytics)
- If any of it is wrong, please redo the installations

6. Resetting helm
- When install tiller too many times
- Console claims that tiller doesnt exist and attempting to install tiller will make the console claim that tiller has already been installed

Reset:
```
kubectl get all --all-namespaces | grep tiller
kubectl delete deployment tiller-deploy -n kube-system
kubectl delete service tiller-deploy -n kube-system
kubectl get all --all-namespaces | grep tiller
```

Init again:

`helm init`


7. Undo reinstallation

- `helm list` : Check the charts you have install
- `helm delete`: Delete the chart
- Delete all configurations (Kubernetes engine > configuration)

> Note that the deployments will be automatically deleted if helm is deleted correctly

# Research

## Seldon + Prometheus

Currently my assumption of the workflow is the seldon-core exposes some metrics which would be scaped by the prometheus which acts as a back-end. The user can then define multiple queries (PromQL) to let the prometheus know what it needs to do and to plot a graph for it. Users can then make use of these graph to be shown in front-end applications such as grafana.


Provided metrics by seldon:
Prediction Requests

`seldon_api_executor_server_requests_seconds_(bucket,count,sum)` : Requests to the service orchestrator from an ingress, e.g. API gateway or Ambassador

`seldon_api_executor_client_requests_seconds_(bucket,count,sum)` : Requests from the service orchestrator to a component, e.g., a model

References:
[Seldon-core metrics docs](https://docs.seldon.io/projects/seldon-core/en/latest/analytics/analytics.html)


## PromQL

There are 4 main parts to the metrics:
1. Name
> E.g varnish_main_client_req


2. One or more labels - KeyValue pairs that differentiates metrics with same names

> A job label will correspond to the scrape config in the prometheus config

3. The value: - Value for the most recent timestamp collected and is a float64

4. Timestamp - The precision in the graph

> Does not appear in the query console

#### Types of metrics:

1. Counters - Cumulative and monotonic

> Values either go up/stay the same / be reset to 0 

2. Gauges - non monotonic

> Either go up or down

e.g `node_memory_utilisation`

3. Histogram - Create multiple series for each metric name

> Creates buckets

4. Summary - Similiar to histogram but creates quantiles (e.g 50th percentile) instead of buckets

#### Query Structure

1. Filter labels
- Can be use to filter the query
  - `=` : equal
  - `!=` : not equal
  - `=~`: matches regex
  - `!~`: doesn't match regex
Label filters go inside the `{}` after metric name.
E.g `Varnish_main_client_req{namespace="seldon-system"}` will only return a metric with with the exact namespace `seldon-system`

`varnish_main_client_req{namespace=~".*3.*",namespace!~".*env4.*"}` will return all varnish_main_client_req metrics with a 3 in their namespace that don’t also contain env4.

2. Range Selectors
- An example range selector: `[1m]`
- Appending this behind our metrics will get multiple values for each timestamp.
- Range-vectors cannot be graphed due to multiple values for each timestamp

> Use range vector to apply a function to it in order to get instant-vector which can be graphed

Functions for working with range-vectors:
- `rate()` - Calculate the per-sec avg rate of increase of the time series over the whole range
- `irate()` - Cal the persec avg increase of time series ***Using only the last two data points in the range***
- `increase()` - Cal the increase in time series per time range selected

Sample taken from website:
![prome_1.png]({{site.baseurl}}/img/prome_1.png)

> Increasing the range will make the graph look less granular

Refer to the reference sheet for more infomation.

Reference:
[PromQL Understanding](https://www.section.io/blog/prometheus-querying/) ,
[PromQL CheatSheet](https://timber.io/blog/promql-for-humans/)


## Prometheus + Grafana
- Grafana makes use of prometheus querying to display how the graph look like

Sample:
![prome_2.PNG]({{site.baseurl}}/img/prome_2.PNG)

As seen in the image, this is accessible via grafana > (Choose any panel) > edit 

Grafana here is making use of a metric through prometheus.

## The entire flow

[Exposing the metrics using executor](https://github.com/SeldonIO/seldon-core/issues/1729)

# My attempts

Using a currently deployed model : (input-output with transformer and model)

Deployment name:
`seldon-model-preprocess-times2-plus2-pod-0-model-transformwjxv5`

Container info:
```
Containers
Name	Status	Image	Restart count	Logs
model	 Running	gcr.io/science-experiments-divya/plus2:input-output	0	View logs
seldon-container-engine	 Running	seldonio/seldon-core-executor:1.1.0	0	View logs
transformer	 Running	gcr.io/science-experiments-divya/times2:input-output	0	View logs
```

Grafana Settings:
![prome_7.PNG]({{site.baseurl}}/img/prome_7.PNG)


## #1 Using postman

Input:
`{"jsonData": {"X" : 4 }}`

This what it is seen in the grafana
![prome_3.PNG]({{site.baseurl}}/img/prome_3.PNG)

Metrics used:
```
sum(rate(seldon_api_executor_server_requests_seconds_count{status!~"5.*"}[1m])) 

sum(rate(seldon_api_executor_server_requests_seconds_count[1m]))
```

## #2 Using executor as recommended by issue

Command:
```
curl -X POST -H 'Content-Type: application/json' -d '{"jsonData": {"X" : 4 }}' http://35.240.217.69:80/seldon/default/seldon-model-preprocess/api/v0.1/predictions
 | jq .
```
> Note I am using Executor

Response:
![prome_4.PNG]({{site.baseurl}}/img/prome_4.PNG)


Command:
`curl -s http://35.240.217.69:80/seldon/default/seldon-model-preprocess/prometheus | grep seldon_api`


Output:

```bash
# HELP seldon_api_executor_client_requests_seconds A histogram of latencies for client calls from executor
# TYPE seldon_api_executor_client_requests_seconds histogram
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.005"} 9
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.01"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.025"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.05"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.1"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.25"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.5"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="1"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="2.5"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="5"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="10"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="+Inf"} 10
seldon_api_executor_client_requests_seconds_sum{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict"} 0.032317318
seldon_api_executor_client_requests_seconds_count{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict"} 10
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.005"} 12
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.01"} 17
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.025"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.05"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.1"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.25"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.5"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="1"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="2.5"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="5"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="10"} 23
seldon_api_executor_client_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="+Inf"} 23
seldon_api_executor_client_requests_seconds_sum{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 0.15186951599999998
seldon_api_executor_client_requests_seconds_count{code="200",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 23
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.005"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.01"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.025"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.05"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.1"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.25"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="0.5"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="1"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="2.5"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="5"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="10"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict",le="+Inf"} 13
seldon_api_executor_client_requests_seconds_sum{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict"} 0.042389772
seldon_api_executor_client_requests_seconds_count{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/plus2",model_name="model",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/predict"} 13
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.005"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.01"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.025"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.05"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.1"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.25"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.5"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="1"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="2.5"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="5"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="10"} 3
seldon_api_executor_client_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="+Inf"} 3
seldon_api_executor_client_requests_seconds_sum{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 0.009852599
seldon_api_executor_client_requests_seconds_count{code="400",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 3
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.005"} 0
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.01"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.025"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.05"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.1"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.25"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="0.5"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="1"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="2.5"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="5"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="10"} 1
seldon_api_executor_client_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input",le="+Inf"} 1
seldon_api_executor_client_requests_seconds_sum{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 0.006282059
seldon_api_executor_client_requests_seconds_count{code="500",deployment_name="seldon-model-preprocess",method="post",model_image="gcr.io/science-experiments-divya/times2",model_name="transformer",model_version="input-output",predictor_name="times2-plus2-pod",predictor_version="",service="/transform-input"} 1
# HELP seldon_api_executor_server_requests_seconds A histogram of latencies for executor server
# TYPE seldon_api_executor_server_requests_seconds histogram
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.005"} 0
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.01"} 9
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.025"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.05"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.1"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.25"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.5"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="1"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="2.5"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="5"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="10"} 10
seldon_api_executor_server_requests_seconds_bucket{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="+Inf"} 10
seldon_api_executor_server_requests_seconds_sum{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 0.07121656
seldon_api_executor_server_requests_seconds_count{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 10
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.005"} 2
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.01"} 5
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.025"} 5
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.05"} 7
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.1"} 13
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.25"} 14
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.5"} 15
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="1"} 16
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="2.5"} 16
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="5"} 16
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="10"} 16
seldon_api_executor_server_requests_seconds_bucket{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="+Inf"} 16
seldon_api_executor_server_requests_seconds_sum{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 1.858107527
seldon_api_executor_server_requests_seconds_count{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 16
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.005"} 0
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.01"} 0
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.025"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.05"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.1"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.25"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="0.5"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="1"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="2.5"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="5"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="10"} 1
seldon_api_executor_server_requests_seconds_bucket{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions",le="+Inf"} 1
seldon_api_executor_server_requests_seconds_sum{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 0.013142297
seldon_api_executor_server_requests_seconds_count{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 1
```

Changes to graph :
![prome_5.PNG]({{site.baseurl}}/img/prome_5.PNG)

Metrics:
```promQL
sum(rate(seldon_api_executor_client_requests_seconds_count{model_name=~"$model_name",model_version=~"$model_version",model_image=~"$model_image",predictor_name=~"$predictor",predictor_version=~"$version"}[5s])) by (model_name,predictor_name,predictor_version,model_image,model_version,service)
```

![prome_6.PNG]({{site.baseurl}}/img/prome_6.PNG)

Sample metric:
```promQL
histogram_quantile(0.5, sum(rate(seldon_api_executor_client_requests_seconds_bucket{service=~"/Predict", model_image=~"$model_image",predictor_name=~"$predictor",predictor_version=~"$version",model_name=~"$model_name",model_version=~"$model_version"}[20s])) by (predictor_name,predictor_version,model_name,model_image,model_version,le))
```

> These are prediction request through executor

References:
[Github Issue](https://github.com/SeldonIO/seldon-core/issues/1729)


## #3 Using commands recommended by seldon docs

```
curl -s http://35.240.217.69:80/seldon/default/seldon-model-preprocess/prometheus | grep seldon_api_executor_server_requests_seconds_count
```

Responses:

```bash
seldon_api_executor_server_requests_seconds_count{code="200",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 10
seldon_api_executor_server_requests_seconds_count{code="400",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 16
seldon_api_executor_server_requests_seconds_count{code="500",deployment_name="seldon-model-preprocess",method="post",predictor_name="times2-plus2-pod",predictor_version="",service="predictions"} 1
```

