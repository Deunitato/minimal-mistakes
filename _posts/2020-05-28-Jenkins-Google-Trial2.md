---
published: false
---
Scapping the pervious, I created a another installation of jenkins following another lab (master)

# Plugins install
- Kubenetes webhook
- google cloud auth
- google cloud build

# Changes compared to the [Guide](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes)
- Use github webhook
- Created a github credentials
- Set number of executors as 3 (manage jenkins > System config)
- created a secret file credentials using the key given in the example


# Original File
```
def project = 'science-experiments-divya'
def  appName = 'seldon-p2t2'
def  feSvcName = "${appName}-plus2"
def  imageTag = "gcr.io/${project}/${appName}:${env.BRANCH_NAME}.${env.BUILD_NUMBER}"
def namespace = "default"
pipeline {

    environment{
     PROJECT_TITLE = 'science-experiments-divya'
     APP_NAME_P2 = 'plus2'
     APP_NAME_T2 = 'times2'

     APP_TAG_P2 = "p2metrics-${env.BUILD_NUMBER}" //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
     APP_TAG_T2 = "input-output" //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER

     DOCKER_FILES_DIR = 'deployment_model'
     CLUSTER = "cluster-1"
     CLUSTER_ZONE = "asia-southeast1-a"
     IMAGE_TAG_P2 = "gcr.io/${PROJECT_TITLE}/${APP_NAME_P2}:${APP_TAG_P2}"
     IMAGE_TAG_T2 = "gcr.io/${PROJECT_TITLE}/${APP_NAME_T2}:${APP_TAG_T2}"

     JENKINS_CRED = "${PROJECT_TITLE}"
     GOOGLE_APPLICATION_CREDENTIALS = credentials('gcp_service_account_key_sentient');
    }


  agent {
    kubernetes {
     label 'plus2-times2-transformer'
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
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG_P2} ./deployment_model"
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG_T2} ./preprocess"
        //    admintag = sh (script:"PYTHONUNBUFFERED=1 gcloud container images list-tags gcr.io/sentient-207310/microservice-apis-auto-admin --limit=${Limit} --sort-by=${Time} --format=${Format}",returnStdout:true)?.trim()
        //     def adminjsonObj = readJSON text: admintag
        //     admintagimage = "${adminjsonObj.tags[0]}"
        //     admintagimagelatest = admintagimage.replaceAll("[^a-zA-Z0-9 ]+","")
        }
      }
    }
    stage('Deploy to k8s deployment-model') {
      // master branch
      when { branch 'master' }
      steps {
        container('kubectl') {
          // Change deployed image in canary to the one we just built
          //sh("sed -i.bak 's#gcr.io/cloud-solutions-images/gceme:1.0.0#${imageTag}#' ./k8s/canary/*.yaml")
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/plus2:p2metrics-0.1#'${IMAGE_TAG_P2}'#' ./k8s/model_deployment.yaml")
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/times2:input-output#'${IMAGE_TAG_T2}'#' ./k8s/model_deployment.yaml")

          sh("kubectl --namespace=${namespace} apply -f k8s/")
        } 
      }
    }
    // stage('Deploy Dev') {
    //   // Developer Branches
    //   when { 
    //     not { branch 'master' } 
    //     not { branch 'canary' }
    //   }     
  }
}
```


