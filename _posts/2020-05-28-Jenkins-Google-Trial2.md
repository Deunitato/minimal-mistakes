---
published: true
---
Scapping the pervious, I created a another installation of jenkins following another lab (master)

# Plugins install
- Kubenetes webhook
- google cloud auth
- google cloud build



# Original File
```
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
        }
      }
    }
    stage('Deploy to k8s deployment-model') {
      when { branch 'master' }
      steps {
        container('kubectl') {
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/plus2:p2metrics-0.1#'${IMAGE_TAG_P2}'#' ./k8s/model_deployment.yaml")
          sh("sed -i.bak 's#gcr.io/science-experiments-divya/times2:input-output#'${IMAGE_TAG_T2}'#' ./k8s/model_deployment.yaml")

          sh("kubectl --namespace=${namespace} apply -f k8s/")
        } 
      }
    }  
  }
}
```

# Steps


1. Clone the repo:
`git clone https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes.git`

2. Move to the directory:
`cd continuous-deployment-on-kubernetes`


3. Create the service account
> This service account is required to run the cloud build in jenkinsfile

`gcloud iam service-accounts create jenkins-sa --display-name "jenkins-sa"`


Assign the permissions:

```
gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/viewer"

gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/source.reader"

gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/storage.admin"

gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/storage.objectAdmin"

gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/cloudbuild.builds.editor"

gcloud projects add-iam-policy-binding $GOOGLE_CLOUD_PROJECT \
    --member "serviceAccount:jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com" \
    --role "roles/container.developer"
```



4. Generate the key
```
gcloud iam service-accounts keys create ~/jenkins-sa-key.json \
    --iam-account "jenkins-sa@$GOOGLE_CLOUD_PROJECT.iam.gserviceaccount.com"
```

> Download the generated key


5. Create the cluster role binding
```
kubectl create clusterrolebinding cluster-admin-binding --clusterrole=cluster-admin --user=$(gcloud config get-value account)
```


6. Installation:

- Ensure you have helm installed: `./helm version`



`./helm install -n cd stable/jenkins -f jenkins/values.yaml --version 1.7.3 --wait`


`kubectl create clusterrolebinding jenkins-deploy --clusterrole=cluster-admin --serviceaccount=default:cd-jenkins`

- Get the password:
`printf $(kubectl get secret cd-jenkins -o jsonpath="{.data.jenkins-admin-password}" | base64 --decode);echo`

7. Create a service loadbalancer deployment

- This is to access the jenkins UI externally

Sample service yaml:

```
apiVersion: v1
kind: Service
metadata:
  name: cd-jenkins-loadbalancer
  labels:
    app.kubernetes.io/name: jenkins
spec:
  type: LoadBalancer
  ports:
    - port: 8080
      targetPort: 8080
  selector:
    app.kubernetes.io/name: jenkins
```
- Creates a load balancer service named `cd-jenkins-loadbalancer`

> Note that the deployment is done in the default namespace



# Changes compared to the [Guide](https://github.com/GoogleCloudPlatform/continuous-deployment-on-kubernetes)
- Use github webhook
- Created a service
- Created a github credentials
- Set number of executors as 3 (manage jenkins > System config)
- created a secret file credentials using the key given in the example (Credentials > Secret key using file)


<iframe src="https://drive.google.com/file/d/1iX87vYWvn7XqvHA2BVD63mBTrJFUjVSg/preview" width="640" height="680"></iframe>
