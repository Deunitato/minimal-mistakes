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

#### Must do when migrating
- Create environment for model path
- Declare user define exceptions
- Create predict_raw method
- Create metadata function (if applicable)

Currently I am integrating `Taxanomy` and `inverse_norm`

## Taxanomy
- Created a Taxanomyapp python file

Full Code:
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

        return jsonify(metadata)


    def predict_raw(self, request):

        # if request.method == "GET":

        #     return jsonify(metadata)
        text = request.json.get("text", None)
        threshold = request.json.get("threshold", 1)
        logger.info(text)

        if (text is None) or (len(text) == 0):
            raise UserCustomException('Bad Request. No data found.',403)

        elif len(text) > max_search_terms:
            raise UserCustomException("Too many search term requested. Max: 20. Received: {}".format(len(text)),403)

        else:
            
            return jsonify(tx.predict(text, threshold))

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


Resources:
[User defined execption for seldon](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/python/python_component.html#user-defined-exceptions), [Tags seldon](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/python/python_component.html#returning-tags), [jsonify vs json](https://stackoverflow.com/questions/7907596/json-dumps-vs-flask-jsonify)

# Others
[Jenkins pipelien repo](https://github.com/Deunitato-sentient/seldon-jenkins-experiement)
