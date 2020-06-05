---
published: true
---
# Things done today
- Seldon python client
- Persistent volume readup
- Deploying persistent volume on k8s - Taxonomy

Issues created today: [Seldon-core-Api-tester command on ambassador](https://github.com/SeldonIO/seldon-core/issues/1914)



# Seldon python client


# Persistent volume readup

## Resources
[{Google cloud persistent volume}](https://cloud.google.com/kubernetes-engine/docs/concepts/persistent-volumes) , [{Kubernetes storage classes}](https://kubernetes.io/docs/concepts/storage/storage-classes/)

# Deploying persistent volume on k8s - Taxonomy

> This is my first time attempting to attach an persistent volume onto the deployment

## Initial try
Steps:
1. Create a disk

Location: compute engine > disk

- Set name
- Change to ssd
- Set zone and region to be similiar to cluster

2. Deploy the `storageclass.yaml`

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

> For type, there is gold/standard/normal

3. Create a bucket
- This is required to store our models

Location: Storage

Name given : science-models-divya

> Note that this is similiar to the yaml file in which we have specified in seldon-deployment.yaml under initContainers: `command: [ '/bin/sh','-c','if [ !$(ls -A /models) ] ; then chmod a+w /models  && gsutil -m cp -r gs://science-models-divya/taxonomy-0.1.0 /models || echo "Not Empty" ; fi' ]`
>
> This command would check if there exist the model, if it doesnt exist, it will copy it from the bucket (Copy once) into the /models folder



