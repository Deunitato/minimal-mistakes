---
published: true
---
# Things done today
- Taxonomy kubernetes migration

Issues created today: [Seldon-core-Api-tester command on ambassador](https://github.com/SeldonIO/seldon-core/issues/1914)



# Seldon python client


# Persistent volume readup

## Resources
[{Google cloud persistent volume}](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes) , [{Kubernetes storage classes}](https://kubernetes.io/docs/concepts/storage/storage-classes/)

# Deploying persistent volume on k8s - Taxonomy

> This is my first time attempting to attach an persistent volume onto the deployment

## Initial try

- This is assuming that the images are already built and pushed to gcr

Steps:
1. Create a disk

Location: compute engine > disk

- Set name
- Change to ssd
- Set zone and region to be similiar to cluster

2. Deploy the storage yaml


In this order:
- Storageclass
- Persistantvolume
- Persistantvolumeclaim

storageclass.yaml:

```yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: gold
provisioner: kubernetes.io/gce-pd
parameters:
  type: pd-ssd
```

Persistantvolume:
- the contents of `pdname` under persistant volume must be the same as the disk that you have created in step 1

Running `kubectl get pv` in this would show the status of this volume as `available`

Persistantvolumeclaim:
```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: taxonomy-0.1.0-pv-claim
spec:
  storageClassName: gold
  accessModes:
    - ReadOnlyMany
  resources:
    requests:
      storage: 1Gi
```
- After applying this, running the `kubectl get pv` will show the `bound` status 

> For type, there is gold/standard/normal

3. Create a bucket
- This is required to store our models

Location: Storage

Name given : science-models-divya

> Note that this is similiar to the yaml file in which we have specified in seldon-deployment.yaml under initContainers: `command: [ '/bin/sh','-c','if [ !$(ls -A /models) ] ; then chmod a+w /models  && gsutil -m cp -r gs://science-models-divya/taxonomy-0.1.0 /models || echo "Not Empty" ; fi' ]`
>
> This command would check if there exist the model, if it doesnt exist, it will copy it from the bucket (Copy once) into the /models folder

4. Copy the models into the bucket

- Upload

> this section is done by my mentor as the file is to big for me to do it 

5. Do dummy init

- Because we only need to write it once and read the other times, we need to create a dummy deployment to push the volumnes into the cloud (since K8 does not have the ability to do readandwriteonce)
- We create a dummy container init, push the volumes, and delete the deployment afterwards

The container init file is similiar to the seldon-deployment.yaml except it does not contain the "readonly = true" parameters inisde its field compared to the seldon-deployment

- Do `kubectl apply -f initcontainer.yaml`

- Check if the volume has been deployed
> Can see a green tick
Location: Workloads > Taxanomy > pods > init-cloud 

- Remove the initcontainer: `kubectl delete -f initcontainer.yaml`

- Deploy the new deployment: `kubectl apply -f seldon_deployment.yaml`

> Check that the pods are working fine

## Errors made

### Code base (changes)
- Missing application code
e.g `raise UserCustomException('Bad Request. No data found.',403)` -> `raise UserCustomException('Bad Request. No data found.',1403,403)`
- Shifted userexception class on top of taxonomy
- function calls inside `taxonomy` classes must be called using `self`
e.g `self.initialise()`
- global terms called inside taxonomy's function must start with the class name
e.g `Taxaonomy.MAX_TERM`
- Logger must be change to not reflect `__name__`
e.g `logger(__name__)` => `logger()`

Reference: [{Logger}](https://www.loggly.com/ultimate-guide/python-logging-basics/) , [{__Name___}](https://www.freecodecamp.org/news/whats-in-a-python-s-name-506262fe61e8/#:~:text=The%20__name__%20variable%20(two%20underscores%20before%20and%20after,a%20module%20in%20another%20script.)

### Building base

#### Sudo apt update error
Error:
![seldon_migration_1.png]({{site.baseurl}}/img/seldon_migration_1.png)

- Receive error: sudo apt update error: “Release file is not yet valid”
- Results in images build partially and images built with no names and tags
Fix: [{issue}](https://askubuntu.com/questions/1096930/sudo-apt-update-error-release-file-is-not-yet-valid)

#### Images build without tags

- Run the command `docker images`
- look for the "built" images with no tags
- Remove using `docker rmi -f <imagetag>`

> Multiple image tags can be stacked by seperating them with spaces
