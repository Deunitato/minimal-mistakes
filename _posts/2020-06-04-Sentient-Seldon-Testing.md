---
published: true
---
# Things covered
- Fix issue of authentication gcloud
- Pytest
- Seldon-api-tester command
- Testing of deployments using pytest

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


## Final solution:

- Update gcloud: `gcloud components update`


## Resources
[Google roles](https://cloud.google.com/container-registry/docs/access-control), [Main authentication doc](https://cloud.google.com/container-registry/docs/advanced-authentication)

# Pytest

# Seldon-core-api-tester
- Using the iris classifier example from the previous day

There are two versions: `seldon-core-api-tester`(1.1.0) and `seldon-core-tester` (v0.3.0)

Base json:


contract.json
```json
{
    "features":[
    {
        "name":"sepal_length",
        "dtype":"FLOAT",
        "ftype":"continuous",
        "range":[4,8]
    },
    {
        "name":"sepal_width",
        "dtype":"FLOAT",
        "ftype":"continuous",
        "range":[2,5]
    },
    {
        "name":"petal_length",
        "dtype":"FLOAT",
        "ftype":"continuous",
        "range":[1,10]
    },
    {
        "name":"petal_width",
        "dtype":"FLOAT",
        "ftype":"continuous",
        "range":[0,3]
    }
    ],
    "targets":[
    {
        "name":"class",
        "dtype":"FLOAT",
        "ftype":"continuous",
        "range":[0,1],
        "repeat":3
    }
    ]
}
```

## Seldon-core-tester (0.3.0)

1. Start the image
```bash
docker run --name "mock_classifier_with_custom_endpoints_rest" -d --rm \
    -e PREDICTIVE_UNIT_SERVICE_PORT=5000 \
    -p 5000:5000 -p 5055:5055 gcr.io/science-experiments-divya/iris_example:0.1
```

2. Execute command 

`seldon-core-tester contract.json 0.0.0.0 5000 -p`

Output:


### Errors:


## Seldon-core-api-tester (1.1.0)
## Resources
<[Iris](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/examples/iris.html?highlight=iris)>,<[seldon-core-tester (0.3)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/examples/custom_endpoints.html)>, <[seldon-core-api-tester (1.1.0)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/workflow/api-testing.html#microservice-api-tester)>


