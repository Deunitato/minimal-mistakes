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

In this case, I used an virtual environment to install my packages

1. Set up venv

- Install it: `sudo apt-get install python3.7-venv`
- Create Venv: `python3 -m venv .`
- Start venv: `source env/bin/activate` or `source bin/activate`

> Depending on where your bin is downloaded




2. Install any dependency

- requests: `pip3 install requests`

3. Create tests

- Note that title/methods should start with `test`

e.g `test_go_method`

My github sample: 

Sample Tutorials:
<[Youtube - pytest starter](https://www.youtube.com/watch?v=wWVXf1WWCl0)> , 

## Resources
<[Python VENV](https://packaging.python.org/tutorials/installing-packages/#creating-virtual-environments)> , <[Json and Python](https://www.w3schools.com/python/python_json.asp)> , <[Asserting json as dicts](https://stackoverflow.com/questions/35030755/assert-json-response)> , <[Converting json as dicts](https://www.geeksforgeeks.org/convert-json-to-dictionary-in-python/)> , <[Sending Headers in Post request](https://stackoverflow.com/questions/9746303/how-do-i-send-a-post-request-as-a-json)>

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
