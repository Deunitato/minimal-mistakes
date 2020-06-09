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
agent any

      stage('Install application dependencies'){
          steps{

                sh """
                apt-get -y update
                apt -y install python3-pip
                apt -y install python-pip
                apt install -y build-essential wget zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev
                wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
                tar -xf Python-3.7.7.tgz
                cd Python-3.7.7
                ./configure --enable-optimizations
                make altinstall
                python3 -m pip install virtualenv
                """

                 sh 'python3.7 --version'
                dir('Alfred'){
                  sh """
                  python3.7 -m pip install -e .
                """
                 }
                sh 'python3.7 -m pip install -r requirements.txt'
                dir('deploy'){
                  sh """
                    python3.7 Alfred_script.py
                    """
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






Resources: [{If-else jenkins}](https://stackoverflow.com/questions/43587964/jenkins-pipeline-if-else-not-working) , [{Change directory Jenkins}](https://stackoverflow.com/questions/52372589/jenkins-pipeline-how-to-change-to-another-folder) , [{file exist jenkins}](https://stackoverflow.com/questions/38534781/check-if-a-file-exists-in-jenkins-pipeline) ,
