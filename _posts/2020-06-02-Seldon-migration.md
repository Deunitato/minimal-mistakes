---
published: false
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