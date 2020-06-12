---
published: true
---
# Things done today
- 2 jenkinsfile in repo
- Deployment of Alfred yaml through jenkins
- pytest test report from jenkins

Working jenkins sample:

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
     IS_PYTHON_EXIST = fileExists 'Python-3.7.7'
     IS_ENV_EXIST = fileExists '.env'
     IS_ALFRED_EXIST = fileExists 'Alfred'

    }
agent any
  stages {
      stage("Fetch Alfred"){
        when { expression { IS_ALFRED_EXIST == 'false' } }
        steps {
                withCredentials([usernamePassword(credentialsId: 'github-divya', usernameVariable: 'USER', passwordVariable: 'PASS')]) 
                {
                    sh """
                    git config user.name "${USER}"
                    git config user.email "${USER}@gmail.com"
                    rm -r alfredtest
                    git clone https://${USER}:${PASS}@github.com/science-experiements-divya/alfredtest.git
                    cp -r alfredtest/Alfred ./
                    """
                } 
            }
      }

      stage('Get python package'){
        when { expression { IS_PYTHON_EXIST == 'false' } }
        steps{
          sh"""
              apt install -y build-essential wget zlib1g-dev libncurses5-dev libgdbm-dev libnss3-dev libssl-dev libreadline-dev libffi-dev
              wget https://www.python.org/ftp/python/3.7.7/Python-3.7.7.tgz
              tar -xf Python-3.7.7.tgz
          """
        }
      }
      
      stage ('Installing python'){
        when { expression { IS_PYTHON_EXIST == 'false' } }
        steps{
            sh """
                  apt-get -y update
                  apt -y install python3-pip
                  apt -y install python-pip
                  cd Python-3.7.7
                  ./configure --enable-optimizations
                  make altinstall
                  """
                  sh 'python3.7 --version'
        }
      }

      stage("Attempt start at virtual environment"){
        when { expression { IS_ENV_EXIST == 'false' } }
        steps{
          sh """
              python3.7 -m pip install virtualenv
              python3.7 -m venv env
          """
        }
      }

      stage('Install application dependencies'){
          steps{
               // dir("env"){
                  sh """
                  virtualenv .env
                  source .env/bin/activate
                  cd Alfred
                  python3.7 -m pip install --ignore-installed -e .
                  cd ..
                  python3.7 -m pip install -r requirements.txt
                  cd deploy
                  python3.7 Alfred_script.py
                """
          }
      }
  }
}
```

# 2 Jenkinsfile in repository

- Was able to do so by specifying the script path in the jenkins configuration




# Fetching through tags

Original file:
```
      stage("Fetch Alfred"){
        when { expression { IS_ALFRED_EXIST == 'false' } }
        steps {
                withCredentials([usernamePassword(credentialsId: 'github-divya', usernameVariable: 'USER', passwordVariable: 'PASS')]) 
                {
                    sh """
                    virtualenv .env
                    source .env/bin/activate
                    git config user.name "${USER}"
                    git config user.email "${USER}@gmail.com"
                    git clone https://${USER}:${PASS}@github.com/science-experiements-divya/alfredtest.git
                    cp -r alfredtest/Alfred ./
                    cd Alfred
                    python3.7 -m pip install --ignore-installed -e .
                    python3.7 -m pip install -r requirements.txt
                    """
                } 
            }
      }
```

## Wget

> doesnt work due to credential issues

