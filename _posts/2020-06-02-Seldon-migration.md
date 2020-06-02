---
published: true
---
# Things done today
- Seldon migration : Taxonomy
- Alfred testing


# Seldon migration
- Current idea is to rid the need for the flask and to use seldon framework instead

## Base code

Base code:
```python
import os
import json
import logging
from flask import Flask, jsonify

from TaxonomyGeneric import Taxonomy

class TaxonomyApp(object):
    """
    Model template. You can load your model parameters in __init__ from a location accessible at runtime
    """
    # ==============================================================================
    # Set up environment
    # ==============================================================================
    logger = logging.getLogger(__name__)

    # ==============================================================================
    # Model directories
    # ==============================================================================
    model_dir = os.environ.get("MODEL_DIR")

    glove_dir = model_dir + "/glove_vec_spacy"

    taxonomy_json_path = model_dir + "/taxonomy.json"

    # ==============================================================================
    # Other variables
    # ==============================================================================
    max_search_terms = int(os.environ.get("MAX_SEARCH_TERMS"))

    # ==============================================================================
    # Instantiate NER object
    # ==============================================================================
    tx = Taxonomy(taxonomy_json_path, embedding_path=glove_dir)

    tx.initialise()

    """
    The field is used to register custom exceptions
    """
    model_error_handler = flask.Blueprint('error_handlers', __name__)

    """
    Register the handler for an exception
    """
    @model_error_handler.app_errorhandler(UserCustomException)
    def handleCustomError(self,error):
        response = jsonify(error.to_dict())
        response.status_code = error.status_code
        return response
    
    def __init__(self, metrics_ok=True, ret_nparray=False, ret_meta=True):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")
        return;

    def tags(self,X):
        # text = request.json.get("text", None)
        # threshold = request.json.get("threshold", 1)
        pass

    def metadata(self):
        """
        Load metadata
        """
        with open("metadata.json", "r") as f:
            metadata = json.load(f)
        return {
            "metadata": jsonify(metadata)
        }

    def predict_raw(self, request):

        # if request.method == "GET":
        myDict = {}

        #     return jsonify(metadata)
        text = request.json.get("text", None)
        threshold = request.json.get("threshold", 1)
        logger.info(text)

        if (text is None) or (len(text) == 0):
            raise UserCustomException('Bad Request. No data found.',403)

        elif len(text) > max_search_terms:
            raise UserCustomException("Too many search term requested. Max: 20. Received: {}".format(len(text)),403)

        else:
            
            return {
                "data": jsonify(tx.predict(text, threshold))
                }

"""
User Defined Exception
"""
class UserCustomException(Exception):

    status_code = 404

    def __init__(self, message, application_error_code,http_status_code):
        Exception.__init__(self)
        self.message = message
        if http_status_code is not None:
            self.status_code = http_status_code
        self.application_error_code = application_error_code

    def to_dict(self):
        rv = {"status": {"status": self.status_code, "message": self.message,
                         "app_code": self.application_error_code}}
        return rv

```

### Code break down and reasoning


```python
	# ==============================================================================
    # Model directories
    # ==============================================================================
    model_dir = os.environ.get("MODEL_DIR")

    glove_dir = model_dir + "/glove_vec_spacy"

    taxonomy_json_path = model_dir + "/taxonomy.json"
```

- Used to initialised the global variable, the persistent volumes as well as paths
- `MODEL_DIR` is retrived from the dockerfile when it is created as an image
- Original code just takes the path from the docker file


#### Code required for seldon for usercustom exception

```python
    """
    The field is used to register custom exceptions
    """
    model_error_handler = flask.Blueprint('error_handlers', __name__)

    """
    Register the handler for an exception
    """
    @model_error_handler.app_errorhandler(UserCustomException)
    def handleCustomError(self,error):
        response = jsonify(error.to_dict())
        response.status_code = error.status_code
        return response
```        
        
#### Class for exception

```python        
"""
User Defined Exception
"""
class UserCustomException(Exception):

    status_code = 404

    def __init__(self, message, application_error_code,http_status_code):
        Exception.__init__(self)
        self.message = message
        if http_status_code is not None:
            self.status_code = http_status_code
        self.application_error_code = application_error_code

    def to_dict(self):
        rv = {"status": {"status": self.status_code, "message": self.message,
                         "app_code": self.application_error_code}}
        return rv

```

- The use to return all the error messages (404/403)

#### Metadata

```python
    def metadata(self):
        """
        Load metadata
        """
        with open("metadata.json", "r") as f:
            metadata = json.load(f)
        return {
            "metadata": jsonify(metadata)
        }
```

- We are going to make use of the metadata endpoint thus we declare the metadata function here
- This remove the need to check if its a GET or POST request


#### Predict

```python
    def predict_raw(self, request):
        text = request.json.get("text", None)
        threshold = request.json.get("threshold", 1)
        logger.info(text)
        
        if (text is None) or (len(text) == 0):
            raise UserCustomException('Bad Request. No data found.',403)
        elif len(text) > max_search_terms:
            raise UserCustomException("Too many search term requested. Max: 20. Received: {}".format(len(text)),403)
        else:
            
            return {
                "data": jsonify(tx.predict(text, threshold))
                }

```

- Since in the [documentation](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/python/api/seldon_core.html?highlight=predict_raw#seldon_core.user_model.SeldonComponent.predict_raw), it is said that predict_raw required user to return a dict instead if json, thus in taxonomy 
- Used raise for exceptions


### Code base: Edit 1
> Included the class taxonomy functions and its helper function

Repo: <[Repo link](https://github.com/Deunitato-sentient/microservice-ai-taxonomy_generic/blob/initial_branch/deploy/TaxonomyGeneric.py)>



# Alfred


## Installation (Without Jupyter)
- Navigate to Alfred directory
- Run command `pip3 install -e ./Alfred`

Copy files

Home dir:
- config-engg.json
- config.yaml

Deploy file:
- Alfred_script.py


## Files

### Config-eng

- ambassador_mappingname: Follow the format `<model_name>_mappings`
- ambassador_customurl: Format is `/microservices/nlp/<modelTitle>/v0.1/getpredictions`
- ambassador_service: `<meta_data_name>/<graph_name>.default:8000`
- model_storage: The name of your container registry

e.g
```
    "ambassador_mappingname": "inverse_norm_mapping",
    "ambassador_customurl": "/microservices/nlp/inverseNorm/v0.1/getpredictions",
    "ambassador_service":"inverse-norm-pod.default:8000",
    "ambassador_timeout":60000
```


```
"model_storage": {"uri":"gs://science-experiments-divya"}}
```

Full sample:

```json
{
    "annotations":
    {
     "project_name":"inverseNorm",
     "deployment_version": "v1",
     "seldon_rest_timeout": "'60000'",
    "seldon_rest_connection_timeout": "'60000'",
    "seldon_grpc_read_timeout": "'60000'",
    "ambassador_mappingname": "inverse_norm_mapping",
    "ambassador_customurl": "/microservices/nlp/inverseNorm/v0.1/getpredictions",
    "ambassador_service":"inverse-norm-pod.default:8000",
    "ambassador_timeout":60000
	
    },
    
"livenessProbe": {
    "failureThreshold": 3,
    "initialDelaySeconds": 100,
    "periodSeconds": 5,
    "successThreshold": 1,
    "timeoutSeconds": 1
  },
  "readinessProbe": {
    "failureThreshold": 3,
    "initialDelaySeconds": 100,
    "periodSeconds": 5,
    "successThreshold": 1,
    "timeoutSeconds": 1
  },
"model_storage": {"uri":"gs://science-experiments-divya"}}
```

> Note that of all the values, the `ambassador_service` must follow the format of the yaml file, for the other config is custom

#### Extras:

Setting up environment
```json
"env": [
    {
      "name": "GUNICORN",
      "value": "4"
    },
    {
      "name": "PORT",
      "value": "5000"
    }
  ],
```

### Config.yaml

Main important changes:
- metadataname
- graphname

> this is important as it affect the other file (json)

Sample:
```yaml
metadata_name : inverse
model_name : inverse_norm
models_used : "yes"
containers :
    - container_name : inverse
      tag_num : 0.1.0
      artefacts : "yes"
      docker_details:
        version : 0.1.0
        ENV:
          - MODEL_NAME : inverse_norm
            SERVICE_TYPE : MODEL
   
graph:
  name: inverse-norm
  type: MODEL
  endpoint:
    type: REST
  children: []
predictor_name: norm-pod
```

### Alfred_script.py

Changes:
- config yaml path

```python
# Saving the path of input config file filled by scientists
config_yaml="/mnt/c/Users/Charlotte/Desktop/NUS DOC/Code/sentient/microservice-ai-inverse_norm/config.yaml"

# Reading the input config file filled by engineering team
# Need to be commented if config-engg.json is not required
with open("/mnt/c/Users/Charlotte/Desktop/NUS DOC/Code/sentient/microservice-ai-inverse_norm/config-engg.json") as f:
  default_dict = json.load(f)
```

- Change the path to match the location

## Running it

Run the command python3 Alfred_scipt.py

Expected output:
- Dockerfile is generated
- Deployment yaml is generated

> Check through ensure its correct

