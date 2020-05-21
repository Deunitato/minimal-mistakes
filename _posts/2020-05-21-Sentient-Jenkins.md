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

5. Set up authentication in jenkins



6. Setting up webhook github

- Ensure the repo is not private
Settings > Webhooks > Payload URL

Enter:
`http://<Jenkins IP>:<Jenkin port>/github-webhook`

e.g `http://34.87.13.193:8080/github-webhook/`

- Ensure that the content-type is application/json

- Check that there is a tick symbol

7. Se

# Problems

## Crumb error
- Keep receiving http request error
- As suggested by [link](https://www.jenkins.io/doc/upgrade-guide/2.176/#SECURITY-626), I installed the plugin "strict crumb" and did the following setting under configure jenkins > Configure global security
![jenkins_3.PNG]({{site.baseurl}}/img/jenkins_3.PNG)

- Tried to apply one setting at a time -> Works

## Webhook is unable to push
- 