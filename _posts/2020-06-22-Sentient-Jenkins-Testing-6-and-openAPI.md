---
published: true
---
# Things done today
- Jenkins: Alfred installtion using tags
- Jenkins: Build number running 


# Alfred installation using tags

## Jenkins
Jenkins configuration
- Using multipipeline job

Config changes:
Under `Branch sources`:

- Source: Github
- Repo used: Alfred_installation
![jenkins-test-9.PNG]({{site.baseurl}}/img/jenkins-test-9.PNG)

Behaviors:
![jenkins-test-10.PNG]({{site.baseurl}}/img/jenkins-test-10.PNG)

Build strategies:
![jenkins-test-11.PNG]({{site.baseurl}}/img/jenkins-test-11.PNG)

### Environment variables
- Ensure that under global configuration, you have a variable name `WORKSPACE_DIR` that links to the home directory of jenkins

e.g `var/JENKINS_HOME`



## Github

### Repository
- Alfred
- jenkinsfile

> Note that Alfred must have requirements.txt within

### Webhook
- URL: `<JENKINSURL>:<PORT>/github-webhook/
- payload: application/json

### Jenkinsfile

This is the working Jenkinsfile:

```
pipeline {

    environment{
     IS_PYTHON_EXIST = fileExists "/${env.WORKSPACE_DIR}/Python-3.7.7"
     IS_ENV_EXIST = fileExists "/${env.WORKSPACE_DIR}/.env"
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
        steps {
                    sh """
                    cd ${env.WORKSPACE_DIR}
                    virtualenv .env
                    source .env/bin/activate
                    cp -r ${env.WORKSPACE}/Alfred ${env.WORKSPACE_DIR}
                    """
        }
      }

      stage("Installing Alfred"){
        steps{
          dir("${env.WORKSPACE_DIR}"){
          sh """
              virtualenv .env
              source .env/bin/activate
              cd Alfred
              python3.7 -m pip install -r requirements.txt
              python3.7 -m pip install --ignore-installed -e .
          """
        }
        }
      }

  }
}
```



## Making a new installation
Imagine you want to update the Alfred installation in the Jenkins cloud

1. Make your changes to Alfred
2. Git commit your changes locally
3. Tag it with the correct tag format using `git tag`
4. Git push to the Alfred Repo with the flag `--tags`
5. Check in Jenkins that your tag is being built

## What it does
- Auto installation of python
- Auto setup of env (if unavailable)
- Auto installation of Alfred within env

# Jenkins: Build number running 
This experiment is to test that we are able to only run a certain stage WHEN the built is initial

Code snippet:
```
        when {     expression {
        return env.BUILD_NUMBER == '1';
        }}
```

- Using this code snippet on top of the stage, we will be able to skip only when the stage is in build phase 1
- This is independent of other jobs

In this example, the stage "Fetching Alfred" has the code snippet above.
Only build `#1` is being executed but in build `#2`, it is not being runned
![jenkins-test-12.PNG]({{site.baseurl}}/img/jenkins-test-12.PNG)

Resources: [{Issue - when condition jenkins}](https://stackoverflow.com/questions/37690920/conditional-step-stage-in-jenkins-pipeline)


