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


