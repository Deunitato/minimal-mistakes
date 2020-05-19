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


3. Start the dashboard on local host:

`kubectl port-forward 
svc/seldon-core-analytics-grafana  3000:80 -n seldon-system`

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
