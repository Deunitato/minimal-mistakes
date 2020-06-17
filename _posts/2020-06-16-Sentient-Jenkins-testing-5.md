---
published: true
---
# Things done today
- Disk creation base on vol folder
- Pytest


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
     GIT_CREDENTIAL_ID = credentials('github-divya')
     GIT_URL = 'science-experiements-divya/taxanomy-trial-jenkins'
     IS_PYTHON_EXIST = fileExists "/${env.WORKSPACE_DIR}/Python-3.7.7"
     IS_ENV_EXIST = fileExists "/${env.WORKSPACE_DIR}/.env"
     IS_ALFRED_EXIST = fileExists "/${env.WORKSPACE_DIR}/Alfred"

    }
agent any
  stages {
      stage('Get python package'){
        when { expression { IS_PYTHON_EXIST == 'false' } }
        steps{
          dir("${env.WORKSPACE_DIR}"){
            sh"""
              ls
              apt-get -y update
              apt install -y build-essential wget zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev
              wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
              tar -xf Python-3.7.7.tgz
              apt -y install python3-pip
              apt -y install python-pip
              cd Python-3.7.7
              ./configure --enable-optimizations
              make altinstall
              """
              sh 'python3.7 --version'
              sh """
                  python3.7 -m pip install virtualenv
                  python3.7 -m venv env
              """
          }
        }
      }

      stage("Attempt start at virtual environment"){
        when { expression { IS_ENV_EXIST == 'false' } }
        steps{
           dir("${env.WORKSPACE_DIR}"){
            sh """
                python3.7 -m pip install virtualenv
                python3.7 -m venv env
            """
           }
        }
      }
      stage("Fetch Alfred"){
        when { expression { IS_ALFRED_EXIST == 'false' } }
        steps {
                withCredentials([usernamePassword(credentialsId: 'github-divya', usernameVariable: 'USER', passwordVariable: 'PASS')]) 
                {
                    sh """
                    cd ${env.WORKSPACE_DIR}
                    virtualenv .env
                    source .env/bin/activate
                    cd ${env.WORKSPACE}
                    git config user.name "${USER}"
                    git config user.email "${USER}@gmail.com"
                    git clone https://${USER}:${PASS}@github.com/science-experiements-divya/alfredtest.git
                    cp -r alfredtest/Alfred ${env.WORKSPACE_DIR}
                    rm -r alfredtest
                    """
                } 
        }
      }

      stage("Installing Alfred"){
        steps{
          dir("${env.WORKSPACE_DIR}"){
          sh """
              virtualenv .env
              source .env/bin/activate
              cd ${env.WORKSPACE}
              cd deploy/scripts
              python3.7 -m pip install -r alfred-requirements.txt
          """
          // if possible, have alfred-requirements stored in alfred
          sh """
              cd Alfred
              python3.7 -m pip install --ignore-installed -e .
          """
        }
        }
      }

      stage('Running Alfred_script'){
          steps{
                dir("${env.WORKSPACE_DIR}"){
                  sh """
                  virtualenv .env
                  source .env/bin/activate
                  cd ${env.WORKSPACE}
                  cd deploy
                  python3.7 scripts/Alfred_script.py
                """
            }
          }
      }

      // stage("Attempt trigger other job"){
      //   steps {
      //     build job: 'microservice/testing-pipeline', propagate: true, wait: true
      //   }
      // }

      stage('Push Branch') {
            steps {

                withCredentials([usernamePassword(credentialsId: 'github-divya', usernameVariable: 'USER', passwordVariable: 'PASS')]) 
                {
                    sh """
                    ls
                    git config user.name "${USER}"
                    git config user.email "${USER}@gmail.com"
                    git config core.excludesFile ~/.gitignore
                    git add .
                    git tag ${APP_TAG}
                    git status
                    git commit -m "update repo"
                    git checkout -b dev
                    git push --force https://${USER}:${PASS}@github.com/science-experiements-divya/snr-microservice-test-pipeline.git dev --tags
                    git checkout master
                    git branch
                    git branch -D dev
                    """
                } 
            }
      }

  }
}
```


## Workflow
1) Check if yaml files exist

- Yes -> Continue step 5)
- No -> Continue step 2)

2) Run Alfred Script

3) Check if theres a volume file

- Yes -> Continue Step 4)
- No -> Continue step 5)

4) Create Volume Container Setup

- Create a disk in gcloud:
`gcloud beta compute disks create disk-1 --project=science-experiments-divya --type=pd-standard --size=500GB --zone=us-central1-a --physical-block-size=4096`
- Run the yaml in the following sequence:
	- Storage class
    - Persistant volume
    - Persistant volume claim
    - initcontainer
- Sleep for 20 s 
> Can be change later, this is to allow the pod to be init
- Delete initcontainer.yaml

5) Run main files
- Docker files
- Seldon_deployment.yaml



We need to follow the file format according to this specification:

1 Image docker:
![jenkins-test-5.PNG]({{site.baseurl}}/img/jenkins-test-5.PNG)


Multiple images docker:
![jenkins-test-6.PNG]({{site.baseurl}}/img/jenkins-test-6.PNG)

# Disk Creation base on vol folder


## Checking if volumefile exist

In environment: `VOLUME_REQUIRED = fileExists "/${env.WORKSPACE}/deploy/volume"`

One GB Creation yields this error:
```
WARNING: You have selected a disk size of under [10GB]. This may result in poor I/O performance. For more information, see: https://developers.google.com/compute/docs/disks#performance.
Created [https://www.googleapis.com/compute/beta/projects/science-experiments-divya/zones/asia-southeast1-a/disks/taxonomy-disk-1].
```

Code snippet of running stage:

```
      stage("Attempt Deploy Volume "){
        when { expression { IS_VOLUME_REQUIRED == 'true' } }
        steps{
          //create Disk image
          container('gcloud') {
                sh """
                 gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
                 """
          sh "gcloud beta compute disks create ${APP_NAME}-disk-1 --project=${PROJECT_TITLE} --type=pd-ssd --size=1GB --zone=asia-southeast1-a --physical-block-size=4096"
          }
          //Deploy files
          dir("${env.WORKSPACE}/deploy/volume"){
            container('kubectl') {
              sh """
            kubectl --namespace=${namespace} apply -f storageclass.yaml
            sleep 5
            kubectl --namespace=${namespace} apply -f persistent_volume.yaml
            sleep 5
            kubectl --namespace=${namespace} apply -f persistentvolumeclaim.yaml
            sleep 5
            kubectl --namespace=${namespace} apply -f initcontainer.yaml
            sleep 40
            kubectl --namespace=${namespace} delete -f initcontainer.yaml
            """
            }
          }
        }
      }
```

- We give sleep so that K8 will have some time to deploy the pods
- Run `kubectl get pv` to check if its properly deployed
- This stage will cover step 4

# Pytest

Plugins:
- Email Extension	
Configuration:
![jenkins-test-7.PNG]({{site.baseurl}}/img/jenkins-test-7.PNG)



Resource:
[Tutorial email](https://www.youtube.com/watch?v=mb8WOHejlT8)

# Resources
[{Jsonpath}](https://kubernetes.io/docs/reference/kubectl/jsonpath/), [{Gcloud disk create}](https://isb-cancer-genomics-cloud.readthedocs.io/en/latest/sections/gcp-info/gcp-info2/Disks.html), [{}]
