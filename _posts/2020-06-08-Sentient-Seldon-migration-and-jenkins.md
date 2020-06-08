---
published: true
---
# Things done today
- Complete inverse_norm migration
- Jenkins pipeline research
	- Virtual env
    - Running of scripts
    - Deploying on to k8s

# Inverse norm migration
## K8s deployment files
- Config.yaml (Alfred)
- Config-engg.json (Alfred)

- initcontainer.yaml
- persistant-volume.yaml
- persistantvolumeclaim.yaml
- seldon-deployment.yaml (generated)
- Storageclass (if this is apply before, no need apply again)

- dockerfile (generated)
- requirement.txt

File Changes:

`initcontainer.yaml`:
- Copy similiar to seldon-deployment.yaml
- Remove all `readonly`

`persistent_volume.yaml`
- change metadata
- change claimref name
- ensure `pdName` under `gcePersistentDisk` reflects the same name as the disk created on k8s

`persistentvolumeclaim.yaml`
- Change the metadata


`requriements.txt`
- add in `seldon-core`

## Steps
1. Create disk in gce
2. `kubectl apply -f persistent_volume.yaml`
	- Check using `kubectl get pv`
3. `kubectl apply -f persistentvolumeclaim.yaml`


# Jenkins pipeline research

## Virtual environment

