---
published: true
---
This is my attempt at creating a jenkins pipeline with the use of gcr google cloud build.

# Plugins
- OAuth Credentials
- Google OAuth Credentials	
- Kubernetes Client API	 
- Kubernetes Credentials	 
- Kubernetes	 
- Google Container Registry Auth	 
- Google Cloud Build	 
- Google Metadata	 
- Google OAuth Credentials	 

# Credentials
- Github
- Google account secret (Service account)


# Attempts

Original jenkinfile

```groovy
node {
    def project = 'science-experiments-divya'
    def app = 'plus2'
    def customImage
    def tag = "p2metrics-${env.BUILD_NUMBER}" //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
    def imageTag = "${project}/${app}:${tag}"
    def registryip = 'https://gcr.io'
    def DOCKER_FILES_DIR = 'deployment_model'

    stage("Clone repo"){
        checkout scm
    }
   

    stage ('initial test'){
        echo 'jenkinsfile is running'
        sh "echo $PATH"
    }

    stage('Initialize'){
        def dockerHome = tool 'myDocker'

        docker{sh "docker ps"}
        env.PATH = "${dockerHome}/bin:${env.PATH}"
    }

    stage('Build Image'){
       customImage = docker.build("${imageTag}")
       customImage.inside {
             sh 'echo "Tests passed"'
         }
    }

    // stage('Test image') {
    //     customImage.inside {
    //         sh 'echo "Tests passed"'
    //     }
    // }
        

    stage 'Docker push'
    //docker.withTool("myDocker"){
        docker.withRegistry("${registryip}", 'gcr:science-experiments-divya') {
        /* Push the container to the custom Registry */
        customImage.push("${tag}") 
        customImage.push("latest") //according to source, push tags is cheap 
     }

     //}
    stage "list pods try one"
    sh("kubectl get pods")

}
```

## Attempt 1 - Creating the image using google cloud


Jenkinsfile:
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
     APP_TAG_T2 = "times2:input-output" //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER

     DOCKER_FILES_DIR = 'deployment_model'
     CLUSTER = "cluster-1"
     CLUSTER_ZONE = "asia-southeast1-a"
     IMAGE_TAG_P2 = "gcr.io/${PROJECT_TITLE}/${APP_NAME_P2}:${APP_TAG_P2}"
     IMAGE_TAG_T2 = "gcr.io/${PROJECT_TITLE}/${APP_NAME_T2}:${APP_TAG_T2}"

     JENKINS_CRED = "${PROJECT_TITLE}"
    }


  agent {
    kubernetes {
      label 'seldon-p2t2'
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
            // sh """
            //     gcloud auth activate-service-account --key-file ${env.GOOGLE_APPLICATION_CREDENTIALS}
            //     """
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t ${IMAGE_TAG_P2} ./deployment_model"
          sh "PYTHONUNBUFFERED=1 gcloud builds submit -t .${IMAGE_TAG_T2} ./preprocess"
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
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/plus2:p2metrics-0.1#'gcr.io/${PROJECT_TITLE}/${APP_NAME_P2}:${IMAGE_TAG_P2}'#' ./k8s/model_deployment.yaml")
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/times2:input-output#'gcr.io/${PROJECT_TITLE}/${APP_NAME_T2}:${IMAGE_TAG_T2}'#' ./k8s/model_deployment.yaml")

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
}
```


References:
[Path to dockerfile](https://stackoverflow.com/questions/58327157/specify-dockerfile-for-gcloud-build-submit), [Sentient example jenkinsfile](https://github.com/sentient-io/platform-flask-flexi/blob/master/prod-gke-yaml/prod-api-auto-deployment.yaml),[Google example jenkinsfile](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes/blob/320bf7b11cc3f4f9d74fe26a173f886574e2ef2f/sample-app/Jenkinsfile)
