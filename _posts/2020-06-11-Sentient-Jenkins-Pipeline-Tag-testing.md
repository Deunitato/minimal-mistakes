---
published: true
---
# Things done
- Jenkins job built base on tags
- Jenkins simple pipeline
- Jenkins 2 source for gitrepo
- 2 Jenkins file: One invoke the other
- Pytest email


# Jenkins job built base on tags

## Plugins must download
- SCM filter branch pr:  [Plugin link](https://plugins.jenkins.io/scm-filter-branch-pr/)
- basic built strategies plugin: [Plugin link](https://github.com/jenkinsci/basic-branch-build-strategies-plugin/blob/master/docs/user.adoc)

## Configuration

Under branch sources:

Behaviours:

![jenkins-test-1.PNG]({{site.baseurl}}/img/jenkins-test-1.PNG)

- This only includes branch name `jenkins-tag-test` and only builts tags with the starting prefix `tag-`
Built strategies:

![jenkins-test-2.PNG]({{site.baseurl}}/img/jenkins-test-2.PNG)

# Jenkins simple pipeline

Tried this: https://dzone.com/articles/adding-a-github-webhook-in-your-jenkins-pipeline

- Webhook does not work

> Dropped

# Jenkins 2 source for gitrepo

```
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
```

- Just clone the repository and the file is "imported"

# Invoking another job from one job

Multipipeline:

Specific branch: Master
```groovy
      stage("Attempt trigger other job"){
        steps {
          build job: 'my-hundredth-pipeline/master', propagate: true, wait: true
        }
      }
```
- This would run another job in jenkins server named "my-hundredth-pipeline" with the branch as master


Single pipeline (No branches):

```
      stage("Attempt trigger other job"){
        steps {
          build job: 'simple-pipeline', propagate: true, wait: true
        }
      }
```
- 'simple-pipeline' is a jenkins job that uses just one pipeline

References: [Issue](https://stackoverflow.com/questions/46471467/jenkins-fails-on-building-a-downstream-job)

