---
published: true
---
## Things did today

- Created a metrics issue
- Create a jenkins pipeline repository
- Seldon testing framework
- Start on seldon migration

# Metrics Issue
- Managed to fix the /metrics endpoint issue by creating a query:
[Metrics endpoint issue](https://github.com/SeldonIO/seldon-core/issues/1901)

The main gist is to
- Do  `-p 5000:5000` and `-p 6000:6000`
- Send a POST request first at port 5000
- Send a GET request at port 6000 after doing a POST

![testframe_2.PNG]({{site.baseurl}}/img/testframe_2.PNG)


# Seldon testing framework

[iago](https://blog.twitter.com/engineering/en_us/topics/infrastructure/2018/iagov2.html)

![testframe_1.PNG]({{site.baseurl}}/img/testframe_1.PNG)

> Taken from seldon blog shows the existance of using iago of testing framework.


[Locust](https://locust.io/)



# Seldon migration



# Others
[Jenkins pipelien repo](https://github.com/Deunitato-sentient/seldon-jenkins-experiement)
