---
published: true
---
Following my internship, I was task to play around in installation of jenkins, a pipelining devops.

# Installation

I followed this setup with reference to this [tutorial](https://devopscube.com/setup-jenkins-on-kubernetes-cluster/)


1. jenkins-deployment.yaml

I pulled the docker image from the latest jenkin file: `docker pull jenkins/jenkins`

```
apiVersion: extensions/v1beta1 # for versions before 1.7.0 use apps/v1beta1
kind: Deployment
metadata:
  name: jenkins-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: jenkins
  template:
    metadata:
      labels:
        app: jenkins
    spec:
      containers:
      - name: jenkins
        image: jenkins/jenkins:latest
        ports:
        - containerPort: 8080
```


2. Service.yaml
```
apiVersion: v1
kind: Service
metadata:
  name: jenkins
spec:
  type: LoadBalancer
  ports:
    - port: 8080
      targetPort: 8080
      nodePort: 30000
  selector:
    app: jenkins
```

3. Password

- To find the password needed to log in, we can run this file
`kubectl logs jenkins-deployment-76c559fb4-5mbm8 --namespace=jenkins`

- Look through the logs for a code like this:

```
*************************************************************

Jenkins initial setup is required. An admin user has been created and a password generated.
Please use the following password to proceed to installation:

30bea689180e46d180fbf68de1090158

This may also be found at: /var/jenkins_home/secrets/initialAdminPassword

*************************************************************
*************************************************************
*************************************************************
```

In this case, `30bea689180e46d180fbf68de1090158` is the password

4. Setup
Using the `http://<node-ip>:30000` we are able to access the UI for jenkins
and start configurations.

- Configure the ip address for jenkins server to be accessible at port `8080`

- Configure the security
	- Manage jenkins > Configure Global Security > CRFC Protection > Check the enable proxy

![jenkins_6.PNG]({{site.baseurl}}/img/jenkins_6.PNG)


## ** DO not go beyond this point > Skip to next step ** ##

Plugins installed:
- Github related authourity and pipeline
- Kubernetes api
- strict crumb issuer

Security:
- Github: 
	- Create new application using [this](https://github.com/settings/applications/new)
    - Followed this:
    ```
    Before configuring the plugin you must create a GitHub application registration.

    Visit https://github.com/settings/applications/new to create a GitHub application registration.
    The values for application name, homepage URL, or application description don't matter. They can be customized however desired.
    However, the authorization callback URL takes a specific value. It must be https://jenkins.example.com/securityRealm/finishLogin where jenkins.example.com is the location of the Jenkins server.

    The important part of the callback URL is /securityRealm/finishLogin

    Finish by clicking Register application.
    The Client ID and the Client Secret will be used to configure the Jenkins Security Realm. Keep the page open to the application registration so this information can be copied to your Jenkins configuration
    ```
- Jenkins:
	- Configure jenkins > Configure global security > Authentication > Global GitHub OAuth Settings
    - Add the client id and client secret as stated in the github application
    
![jenkins_1.PNG]({{site.baseurl}}/img/jenkins_1.PNG)

![jenkins_2.PNG]({{site.baseurl}}/img/jenkins_2.PNG)

> It is found that creating app is not needed, just jump straight into setting up authentication under jenkins

5. Set up authentication in jenkins
- From the side panel > Credentials > System 
- Click on Global Credential
- From the side panel > add credentials
- Type> Kind with password:
	- Username : github username
    - password: github password
    - type: `github`


6. Setting up webhook github

- Ensure the repo is not private
Settings > Webhooks > Payload URL

Enter:
`http://<Jenkins IP>:<Jenkin port>/github-webhook`


e.g `http://34.87.13.193:8080/github-webhook/`

- Ensure that the content-type is application/json

- Check that there is a tick symbol

7. Creation of new project (Jenkins)
- Set up new project
- enter new name > Type: pipeline

#### Settings:
- General > github prkject > project url
	- e.g `https://github.com/Deunitato-sentient/jenkins-pipeline-examples`
- Pipeline > Definition: `pipeline script from scm`
	- SCM: `git`
    - Repository url (ending with .git) e.g `https://github.com/Deunitato-sentient/jenkins-pipeline-examples.git`
    - Credential: The github account set up

8. Build the pipeline
Configure > Build trigger > GitHub hook trigger for GITScm polling
- At the side panel > Build now
![jenkins_5.PNG]({{site.baseurl}}/img/jenkins_5.PNG)

### Resource
[IBM tutorial](https://developer.ibm.com/technologies/devops/tutorials/configure-a-cicd-pipeline-with-jenkins-on-kubernetes/)
[Setup tutorial - devopscube](https://devopscube.com/setup-jenkins-on-kubernetes-cluster/)
[Setup tutorial - phoenixnap ](https://phoenixnap.com/kb/how-to-install-jenkins-kubernetes)

# Problems

## Crumb error
- Keep receiving http request error
- As suggested by [link](https://www.jenkins.io/doc/upgrade-guide/2.176/#SECURITY-626), I installed the plugin "strict crumb" and did the following setting under configure jenkins > Configure global security
![jenkins_3.PNG]({{site.baseurl}}/img/jenkins_3.PNG)

- Tried to apply one setting at a time -> Works

## Webhook is unable to push
- Possibly repo is in private
- At the bottom of webhook, there is an option to redeliever

# Getting it to work

- The way jenkins work is that the webhook in the github repo will detect any `Jenkinsfile` located inside the repo and will execute it. 

Plugins installed:
- Blue ocean
- google container auth
- docker build step
- [Kubernetes CLI](https://github.com/jenkinsci/kubernetes-cli-plugin) (And some related kubernetes plugins)
- [Google Auth link 1](https://github.com/jenkinsci/google-oauth-plugin/blob/develop/docs/home.md), [Google auth link2](https://plugins.jenkins.io/google-container-registry-auth/)



## Docker CLI

### Failed attempt 1


Jenkinsfile:

```
node {
    def project = 'science-experiments-divya'
    def app = 'plus2'
    def tag = 'p2metrics-0.1' //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
    def imageTag = 'gcr.io/${project}/${app}:${tag}'
    checkout scm

    //stages {
        // stage('Build docker image') {
        //     steps {
        //         echo 'Building..'
        //         echo "Running the script now"
        //         docker.withRegistry("https://gcr.io", "gcr:credential-id") {
        //         sh 'docker build --tag gcr.io/science-experiments-divya/plus2:p2metrics-0.1 .'
        //         sh 'docker push gcr.io/science-experiments-divya/plus2:p2metrics-0.1'
        //         }
            
                
        //     }
        // }
        stage 'initial test'
        echo 'jenkinsfile is running'

        stage 'Build test'
        sh("docker build -t ${imageTag} .")

        stage 'Run Go test'
        sh("docker run ${imageTag} go test")

        stage 'push to gcr'
        sh("gcloud docker --push ${imageTag}")

        
        stage "list pods try one"
        sh("kubectl get pods")

}
```

> /var/jenkins_home/workspace/multibranch-pipeline-1_master@tmp/durable-fa33915e/script.sh: 1: /var/jenkins_home/workspace/multibranch-pipeline-1_master@tmp/durable-fa33915e/script.sh: docker: not found

### Failed Attempt 2

Code:
```
pipeline {
    agent { dockerfile true }
    stages {
        stage('Test') {
            steps {
                sh 'node --version'
                sh 'svn --version'
            }
        }
    }
}
```

Error:

```
Push event to branch master
04:10:23 Connecting to https://api.github.com using science-experiements-divya/******
Obtained Jenkinsfile from c2f21fcfa5cd70453d53bda353afc5429b1b5e99
Running in Durability level: MAX_SURVIVABILITY
[Pipeline] Start of Pipeline
[Pipeline] node
Running on Jenkins in /var/jenkins_home/workspace/multibranch-pipeline-1_master
[Pipeline] {
[Pipeline] stage
[Pipeline] { (Declarative: Checkout SCM)
[Pipeline] checkout
using credential github
 > git rev-parse --is-inside-work-tree # timeout=10
Fetching changes from the remote Git repository
 > git config remote.origin.url https://github.com/science-experiements-divya/seldon-trial-experiements.git # timeout=10
Fetching without tags
Fetching upstream changes from https://github.com/science-experiements-divya/seldon-trial-experiements.git
 > git --version # timeout=10
using GIT_ASKPASS to set credentials 
 > git fetch --no-tags --progress -- https://github.com/science-experiements-divya/seldon-trial-experiements.git +refs/heads/master:refs/remotes/origin/master # timeout=10
Checking out Revision c2f21fcfa5cd70453d53bda353afc5429b1b5e99 (master)
 > git config core.sparsecheckout # timeout=10
 > git checkout -f c2f21fcfa5cd70453d53bda353afc5429b1b5e99 # timeout=10
Commit message: "Add jenkinsfile"
 > git rev-list --no-walk a416108802ef256178df2bd328e72b4d3c0f9a81 # timeout=10
[Pipeline] }
[Pipeline] // stage
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withDockerRegistry
$ docker login -u _token -p ******** https://gcr.io/
[Pipeline] // withDockerRegistry
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline

GitHub has been notified of this commit’s build result

java.io.IOException: error=2, No such file or directory
	at java.lang.UNIXProcess.forkAndExec(Native Method)
	at java.lang.UNIXProcess.<init>(UNIXProcess.java:247)
	at java.lang.ProcessImpl.start(ProcessImpl.java:134)
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:1029)
Caused: java.io.IOException: Cannot run program "docker": error=2, No such file or directory
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:1048)
	at hudson.Proc$LocalProc.<init>(Proc.java:252)
	at hudson.Proc$LocalProc.<init>(Proc.java:221)
	at hudson.Launcher$LocalLauncher.launch(Launcher.java:936)
	at hudson.Launcher$ProcStarter.start(Launcher.java:454)
	at hudson.Launcher$ProcStarter.join(Launcher.java:465)
	at org.jenkinsci.plugins.docker.commons.impl.RegistryKeyMaterialFactory.materialize(RegistryKeyMaterialFactory.java:101)
	at org.jenkinsci.plugins.docker.workflow.AbstractEndpointStepExecution2.doStart(AbstractEndpointStepExecution2.java:53)
	at org.jenkinsci.plugins.workflow.steps.GeneralNonBlockingStepExecution.lambda$run$0(GeneralNonBlockingStepExecution.java:77)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
Finished: FAILURE
```


## Kubernetes CLI

#### Generating credentials
```
# Create a ServiceAccount named `jenkins-robot` in a given namespace.
$ kubectl -n <namespace> create serviceaccount jenkins-robot

# The next line gives `jenkins-robot` administator permissions for this namespace.
# * You can make it an admin over all namespaces by creating a `ClusterRoleBinding` instead of a `RoleBinding`.
# * You can also give it different permissions by binding it to a different `(Cluster)Role`.
$ kubectl -n <namespace> create rolebinding jenkins-robot-binding --clusterrole=cluster-admin --serviceaccount=<namespace>:jenkins-robot

# Get the name of the token that was automatically generated for the ServiceAccount `jenkins-robot`.
$ kubectl -n <namespace> get serviceaccount jenkins-robot -o go-template --template='{{range .secrets}}{{.name}}{{"\n"}}{{end}}'


# Retrieve the token and decode it using base64.
$ kubectl -n <namespace> get secrets jenkins-robot-token-d6d8z -o go-template --template '{{index .data "token"}}' | base64 -d

```

### Attempt 3

Code:
```
node {
    def project = 'science-experiments-divya'
    def app = 'plus2'
    def tag = 'p2metrics-0.1' //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
    def imageTag = '${project}/${app}:${tag}'
    def registryip = 'https://gcr.io'
    checkout scm

        stage 'initial test'
        echo 'jenkinsfile is running'
        

        stage 'Docker test'
        sh("docker build -t ${imageTag} .")
        docker.withRegistry("${registryip}", 'gcr:google-container-registry') {

        def customImage = docker.build("${imageTag}")

        /* Push the container to the custom Registry */
        customImage.push() }

        
        stage "list pods try one"
        sh("kubectl get pods")

}
```

Return:
```
/var/jenkins_home/workspace/multibranch-pipeline-1_master@tmp/durable-42233bfe/script.sh: 1: /var/jenkins_home/workspace/multibranch-pipeline-1_master@tmp/durable-42233bfe/script.sh: docker: not found
```


Resources:
[Stackoverflow issue #1](https://stackoverflow.com/questions/54573068/pushing-docker-image-through-jenkins), [Google container registry auth Plugin](https://plugins.jenkins.io/google-container-registry-auth/), [Official Jenkins docker page](https://www.jenkins.io/doc/book/pipeline/docker/#custom-registry)


#### Installed the following plugins:
- Docker API	 
- Docker	 
- CloudBees Docker Build and Publish Loading plugin extensions	 

## Attempt #4

```
node {
    def project = 'science-experiments-divya'
    def app = 'plus2'
    def tag = 'p2metrics-0.1' //change this to $env.BRANCH_NAME or $env.BUILD_NUMBER
    def imageTag = '${project}/${app}:${tag}'
    def registryip = 'https://gcr.io'
    def DOCKER_FILES_DIR = 'deployment_model'
    checkout scm

        stage 'initial test'
        echo 'jenkinsfile is running'

        stage('Initialize'){
        def dockerHome = tool 'myDocker'
        env.PATH = "${dockerHome}/bin:${env.PATH}"
        }
        

        stage 'Docker test'
        docker.withRegistry("${registryip}", 'gcr:science-experiments-divya') {

        def customImage = docker.build("${imageTag}")

        /* Push the container to the custom Registry */
        customImage.push() }

        
        stage "list pods try one"
        sh("kubectl get pods")

}
```

> I also tried to change the path and they gave the same error

Resources:
[Stackoverflow suggestion](https://stackoverflow.com/questions/44850565/docker-not-found-when-building-docker-image-using-docker-jenkins-container-pipel), [subdirectory path suggestion](https://stackoverflow.com/questions/54309406/how-build-a-dockerfile-in-a-subdirectory-using-a-jenkinsfile)


Output:
```
Using the ‘stage’ step without a block argument is deprecated
Entering stage Docker test
Proceeding
[Pipeline] withEnv
[Pipeline] {
[Pipeline] withDockerRegistry
$ docker login -u _token -p ******** https://gcr.io
[Pipeline] // withDockerRegistry
[Pipeline] }
[Pipeline] // withEnv
[Pipeline] }
[Pipeline] // node
[Pipeline] End of Pipeline

GitHub has been notified of this commit’s build result

java.io.IOException: error=2, No such file or directory
	at java.lang.UNIXProcess.forkAndExec(Native Method)
	at java.lang.UNIXProcess.<init>(UNIXProcess.java:247)
	at java.lang.ProcessImpl.start(ProcessImpl.java:134)
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:1029)
Caused: java.io.IOException: Cannot run program "docker": error=2, No such file or directory
	at java.lang.ProcessBuilder.start(ProcessBuilder.java:1048)
	at hudson.Proc$LocalProc.<init>(Proc.java:252)
	at hudson.Proc$LocalProc.<init>(Proc.java:221)
	at hudson.Launcher$LocalLauncher.launch(Launcher.java:936)
	at hudson.Launcher$ProcStarter.start(Launcher.java:454)
	at hudson.Launcher$ProcStarter.join(Launcher.java:465)
	at org.jenkinsci.plugins.docker.commons.impl.RegistryKeyMaterialFactory.materialize(RegistryKeyMaterialFactory.java:101)
	at org.jenkinsci.plugins.docker.workflow.AbstractEndpointStepExecution2.doStart(AbstractEndpointStepExecution2.java:53)
	at org.jenkinsci.plugins.workflow.steps.GeneralNonBlockingStepExecution.lambda$run$0(GeneralNonBlockingStepExecution.java:77)
	at java.util.concurrent.Executors$RunnableAdapter.call(Executors.java:511)
	at java.util.concurrent.FutureTask.run(FutureTask.java:266)
	at java.util.concurrent.ThreadPoolExecutor.runWorker(ThreadPoolExecutor.java:1149)
	at java.util.concurrent.ThreadPoolExecutor$Worker.run(ThreadPoolExecutor.java:624)
	at java.lang.Thread.run(Thread.java:748)
```






## Resources:
[Jenkinsfile to GCR](https://stackoverflow.com/questions/54573068/pushing-docker-image-through-jenkins)

Possible google provided installation:
[Install](https://github.com/GoogleCloudPlatform/click-to-deploy/blob/master/k8s/jenkins/README.md), [Cloud marketplace](https://console.cloud.google.com/marketplace/kubernetes/config/google/jenkins?version=2.190&project=science-experiments-divya&authuser=3&folder=true&organizationId=true)