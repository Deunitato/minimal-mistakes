---
published: true
---
# Things covered
- Fix issue of authentication gcloud
- Pytest
- Seldon-api-tester command

# Gcloud authentication
Issue:
Not able to push docker image despite being authenticated and configured

## Things tried and failed:
- Gcloud init  > Reconfigure
- Create new service account > imput key > `gcloud auth activate-service-account ACCOUNT --key-file=KEY-FILE`
- Restart computer
- gave permission to my account with "storage admin" role
- using `gcloud auth login`
- Delete config file in `~/.docker` > re init using `gcloud auth configure-docker`
- Update gcloud: `gcloud components update`
- Reinstalling gcloud


## Resources
[Google roles](https://cloud.google.com/container-registry/docs/access-control), [Main authentication doc](https://cloud.google.com/container-registry/docs/advanced-authentication)

# Pytest

# Seldon-core-api-tester
- Using the iris classifier example from the previous day

There are two versions: `seldon-core-api-tester`(1.1.0) and `seldon-core-tester` (v0.3.0)


## Resources
<[Iris](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/examples/iris.html?highlight=iris)>,<[seldon-core-tester (0.3)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/examples/custom_endpoints.html)>, <[seldon-core-api-tester (1.1.0)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/workflow/api-testing.html#microservice-api-tester)>


