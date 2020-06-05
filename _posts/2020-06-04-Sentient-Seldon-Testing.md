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

4. Run test file
- `pytest <FILENAME>`

For example, `<FILENAME>` can be `/iris_classifier` in the provided example

My github sample: [link](https://github.com/Deunitato-sentient/snr-pytest-samples)

Sample Tutorials:
<[Youtube - pytest starter](https://www.youtube.com/watch?v=wWVXf1WWCl0)> , 

## Resources
<[Python VENV](https://packaging.python.org/tutorials/installing-packages/#creating-virtual-environments)> , <[Json and Python](https://www.w3schools.com/python/python_json.asp)> , <[Asserting json as dicts](https://stackoverflow.com/questions/35030755/assert-json-response)> , <[Converting json as dicts](https://www.geeksforgeeks.org/convert-json-to-dictionary-in-python/)> , <[Sending Headers in Post request](https://stackoverflow.com/questions/9746303/how-do-i-send-a-post-request-as-a-json)>

# Seldon-core-api-tester
- Using the iris classifier example from the previous day

There are two versions: `seldon-core-api-tester`(1.1.0) and `seldon-core-tester` (v0.3.0)

Both version are packaged within the seldon-core that you have installed using pip

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

- Ensure that you are in the same directory as the contract.json

1. Start the image
```bash
docker run --name "mock_classifier_with_custom_endpoints_rest" -d --rm \
    -e PREDICTIVE_UNIT_SERVICE_PORT=5000 \
    -p 5000:5000 -p 5055:5055 gcr.io/science-experiments-divya/iris_example:0.1
```


2. Execute command 

`seldon-core-tester contract.json 0.0.0.0 5000 -p`

Output:

```
SENDING NEW REQUEST:
{'meta': {}, 'data': {'names': [u'sepal_length', u'sepal_width', u'petal_length', u'petal_width'], 'ndarray': [[7.77, 4.122, 2.927, 1.133]]}}
RECEIVED RESPONSE:
{u'meta': {}, u'data': {u'names': [u't:0', u't:1', u't:2'], u'ndarray': [[0.8892459987401032, 0.11072983490657469, 2.416635332199314e-05]]}}
()
Time 0.013571023941
```

- Prometheus command

`!curl "http://localhost:5055/prometheus_metrics"`


Output:
```
curl  -s http://127.0.0.1:5000/api/v1.0/predictions -H "Content-Type: application/json" -d '{"data":{"ndarray":[[5.964,4.006,2.081,1.031]]}}' "http://localhost:5055/prometheus_metrics"
{"data":{"names":["t:0","t:1","t:2"],"ndarray":[[0.9548873249364059,0.04505474761562512,5.7927447968953825e-05]]},"meta":{}}
```

3. Closing the session
`docker rm -v "mock_classifier_with_custom_endpoints_rest" --force`

> Note: using `sudo fuser -k 5000/tcp` does not work

## Seldon-core-api-tester (1.1.0)

Check existance:
`seldon-core-microservice --help`

1. Start the docker

`docker run --name "my_model" -d --rm -p 5000:5000  gcr.io/science-experiments-divya/iris_example:0.1`

2. Execute the command

- Execute directly

> Ensure that before using this command, you are in the environement you have installed dependencies on (ie. joblib)

`seldon-core-microservice IrisClassifier REST`




## Resources
<[Iris](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/examples/iris.html?highlight=iris)>,<[seldon-core-tester (0.3)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/examples/custom_endpoints.html)>, <[seldon-core-api-tester (1.1.0)](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/workflow/api-testing.html#microservice-api-tester)>
