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