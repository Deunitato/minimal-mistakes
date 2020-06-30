---
published: true
---
Theres a problem with my Jenkins pipeline and I was unable to run the nodes in the same workspace. So this entire post is dedicated to the solving of that problems

# Labels

[=== Suggestion source ====]{https://stackoverflow.com/questions/59425623/how-to-copy-a-file-from-jenkins-agent-node-to-a-remote-server-using-jenkinsfile}

## Comments:
This method is not possible as label signifies the configured node under (manage jenkins > Configure nodes) 
However, our kubernetes agent is using a different type of node that is not specifically configured inside our jenkins server (google cloud).


# Same workspace for multiple jobs
[=== Suggestion source ====](https://stackoverflow.com/questions/21520475/same-workspace-for-multiple-jobs)

## Comments:
- The nodes are sharing different workspace, we cannot clone nor copy if the workspace is different in the first place
- Original method is to cd to that workspace and we can just copy
- Plugin doesnt allow us to check our other node's workspace
- Kubernetes has a restriction where we are not allowed to cd into the main workspace

# Clone workspace scm plugin

## Comments: 
- Doesnt work ob pipeline (from issue)

# Archiving
Plugins:
- Copy to archive plugin

Changes:
- Seperate build steps to two jobs

# Stash (Working solution)

- Just stash it in the previous node
- Unstash it in the new node


Code sample:
Any node:
```
stage("stashing"){
  steps{
    stash includes: 'deploy/*', name: 'deploy'
  }
}
```

- This will stash all files under deploy in the name deploy

Kubernetes node:

``` groovy
 stage('Build and push image with Container Builder') {
    steps {
      container('gcloud') {
          unstash 'deploy'
          sh """
              ls
              gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
              """
        sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG} ./deploy"
      }
    }
  }
```


Final working Jenkinsfike:

```

def namespace = "default"
pipeline {
  environment{

    // Fill up these


    /*
    Project details

    PROJECT_TITLE: Name of your parent project

    */
    PROJECT_TITLE = 'science-experiments-divya'

    /* 
     == Jenkins credentials ==

    GOOGLE_APPLICATION_CREDENTIALS: Service account credentials of the gcr 
    GOOGLE_BUCKET_CREDENTIALS: Service account credentials of the bucket stated in ORIGIN_BUCKET

    */
    JENKINS_CRED = "${PROJECT_TITLE}"
    GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp_service_account_key_sentient');
    GOOGLE_BUCKET_CREDENTIALS = credentials('gcp_service_account_key_bucket');

    /*
    == Project details for disk creation ==
    
      APP_NAME: Used for disk name
      CLUSTER_ZONE: Used for disk creation
      DISK_SIZE: in GB
      TYPE_OF_DISK : pd-ssd  or pd-hdd

    */
    APP_NAME = 'taxonomy'
    CLUSTER_ZONE = "asia-southeast1-a"
    DISK_SIZE = '10'  
    TYPE_OF_DISK = 'pd-ssd'


    /*
    == Details for image building ==
    
      IMAGE_TAG: Ensure this is the same as the one in your config.yaml

    */
    IMAGE_TAG = "gcr.io/${PROJECT_TITLE}/${APP_NAME}:${APP_TAG}"


    /*
    == Bucket copy details ==
 
      ORIGIN_BUCKET: Where the models are stored (path)
      TARGET_BUCKET: Where the models are to be stored (path)

    */
    ORIGIN_BUCKET = "gs://sentient-science-models/nlp/taxonomy"
    TARGET_BUCKET = "gs://science-models-divya/"

    // Do not touch these
    
    /*
     == File checks ==
    */
    IS_PYTHON_EXIST = fileExists "/${env.WORKSPACE_DIR}/Python-3.7.7"
    IS_ENV_EXIST = fileExists "/${env.WORKSPACE_DIR}/.env"
    IS_ALFRED_EXIST = fileExists "/${env.WORKSPACE_DIR}/Alfred"
    IS_VOLUME_REQUIRED = fileExists "/${env.WORKSPACE}/deploy/volume"

  }
  agent {
      kubernetes {
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

  /*  = Working =
    - Assumption that Alfred folder exist in the main home directoy
    - Copy the script into the deploy folder
    - Activate the environment
    - Run the script
    - Delete the script
  */
  stage('Execute Alfred_script'){
    agent any
    steps{
          dir("${env.WORKSPACE_DIR}"){
            sh """
            cp Alfred/Alfred/script/Alfred_script.py ${env.WORKSPACE}/deploy
            virtualenv .env
            source .env/bin/activate
            cd ${env.WORKSPACE}
            cd deploy
            python3.7 Alfred_script.py
            rm Alfred_script.py
          """
      }
        stash includes: 'deploy/*', name: 'deploy'
    }
  }

/**                git checkout master
                git branch
                git branch -D dev */
  // stage('Push to master branch repository') {
  //       agent any
  //       steps {
  //           withCredentials([usernamePassword(credentialsId: 'github-divya', usernameVariable: 'USER', passwordVariable: 'PASS')]) 
  //           {
  //               sh """
  //               ls
  //               git config user.name "${USER}"
  //               git config user.email "${USER}@gmail.com"
  //               git config core.excludesFile ~/.gitignore
  //               git add .
  //               git status
  //               git commit -m "Push installed files"
  //               git checkout -b jenkins-files
  //               git push --force https://${USER}:${PASS}@github.com/science-experiements-divya/snr-microservice-test-pipeline.git jenkins-files
  //               git checkout master
  //               git branch -d jenkins-files

  //     """
  //           } 
  //       }
  // }

  stage('Build and push image with Container Builder') {
    steps {
      container('gcloud') {
          unstash 'deploy'
          sh """
              ls
              gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
              """
        sh "PYTHONUNBUFFERED=1 gcloud builds submit --timeout=2h15m5s -t ${IMAGE_TAG} ./deploy"
      }
    }
  }

  stage('Copy model files to bucket'){ 
    steps{
        // Copy model files
        container('gcloud') {
            sh """
              gcloud auth activate-service-account --key-file ${env.GOOGLE_BUCKET_CREDENTIALS}
              """
            sh "gsutil -m cp -r ${ORIGIN_BUCKET} ${TARGET_BUCKET}"
        }
    }
  }


  stage("Create Disk"){
    when { expression { IS_VOLUME_REQUIRED == 'true' && env.BUILD_NUMBER == '1'} }
    steps{
      //create Disk image
      container('gcloud') {
            sh """
              gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
              """
      sh "gcloud beta compute disks create ${APP_NAME}-disk-1 --project=${PROJECT_TITLE} --type=${TYPE_OF_DISK} --size=${DISK_SIZE}GB --zone=${CLUSTER_ZONE} --physical-block-size=4096"
      }
    }
  }
// && env.BUILD_NUMBER == '1'
  stage("Deploy volume"){
    when { expression { IS_VOLUME_REQUIRED == 'true'} }
    steps{
      //Deploy files
      sh 'ls'
      dir("${env.WORKSPACE}/deploy/volume"){
        container('kubectl') {
          sh """
        kubectl --namespace=${namespace} apply -f storageclass.yaml
        sleep 5
        kubectl --namespace=${namespace} apply -f persistent_volume.yaml
        sleep 5
        kubectl --namespace=${namespace} apply -f persistentvolumeclaim.yaml
        sleep 5
        """
        }
        //deploy initcontainer
        container('kubectl'){
          sh"""
        kubectl --namespace=${namespace} apply -f initcontainer.yaml
        sleep 40
        kubectl --namespace=${namespace} delete -f initcontainer.yaml
          """
        }
      }
    }
  }


  stage('Deploy to model') {
 
    steps {
      container('kubectl') {
        sh("kubectl --namespace=${namespace} apply -f deploy/seldon_deployment.yaml")
      } 
    }
  } 

    // stage("Attempt trigger other job"){
    //   steps {
    //     build job: 'microservice/testing-pipeline', propagate: true, wait: true
    //   }
    // }

}}
``