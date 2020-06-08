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

Original Jenkinsfile:
```

def namespace = "default"
pipeline {

    environment{
     PROJECT_TITLE = 'science-experiments-divya'
     APP_NAME = 'taxonomy'

     APP_TAG = "0.1.${env.BUILD_NUMBER}" //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
     CLUSTER = "cluster-1"
     CLUSTER_ZONE = "asia-southeast1-a"
     IMAGE_TAG = "gcr.io/${PROJECT_TITLE}/${APP_NAME}:${APP_TAG}"

     JENKINS_CRED = "${PROJECT_TITLE}"
     GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp_service_account_key_sentient');
    }


  agent {
    kubernetes {
     label 'taxonomy-1'
      defaultContainer 'jnlp'
      yaml """
apiVersion: v1
kind: Pod
metadata:
  labels:
    component: ci
spec:
  # Use service account that can deploy to all namespaces
  serviceAccountName: cd-jenkins
  containers:
  - name: gcloud
    image: gcr.io/cloud-builders/gcloud
    command:
    - cat
    tty: true
  - name: kubectl
    image: gcr.io/cloud-builders/kubectl
    command:
    - cat
    tty: true
"""
  }
}
  stages {
    stage('Build and push image with Container Builder') {
      steps {
        container('gcloud') {
             sh """
                 gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
                 """
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG} ./deploy"
        }
      }
    }
    stage('Deploy to k8s deployment-model') {
      // master branch
      when { branch 'master' }
      steps {
        container('kubectl') {
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/taxonomy:0.1.7#'${IMAGE_TAG}'#' ./k8s/*.yaml")

          sh("kubectl --namespace=${namespace} apply -f k8s/")
        } 
      }
    }  
  }
}
```


## Private repo
Follow the github setup and add a github credentials to access a private repository

## Virtual environment

- Installed [pyenv plugin](https://www.jenkins.io/doc/pipeline/steps/pyenv-pipeline/)
- Python
- Python Wrapper
- docker



Resources: [{If-else jenkins}](https://stackoverflow.com/questions/43587964/jenkins-pipeline-if-else-not-working) , [{Change directory Jenkins}](https://stackoverflow.com/questions/52372589/jenkins-pipeline-how-to-change-to-another-folder) , [{file exist jenkins}](https://stackoverflow.com/questions/38534781/check-if-a-file-exists-in-jenkins-pipeline) , 
