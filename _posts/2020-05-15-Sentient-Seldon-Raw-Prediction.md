---
published: false
---
Here I am going to do the testing of the raw prediction method that is avaliable in Seldon

# Base Code 

plus2.py (model) - v1.0
```
import logging
import base64
class plus2(object):
   
    def predict(self, X, feature_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info("Char's log (model): " + str(X))
        #myStrin = X.decode() + " appended byte string in times2"
        value = X.get("X") + 2
        
        return "The return value is " + str(value)

```

times2.py (transformer) - v1.0:
```
import logging
class times2(object):

    def __init__(self):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")

        return


    def transform_input(self, X, feature_names):
        logging.info("Char's log (transform): " + str(X))
        value = X.get("X") * 2
        mydict = {} 
        mydict["X"] = value
        return mydict


```

yaml file:
```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-example
  labels:
    model_name: times2-plus2-raw
    api_type: microservice
    microservice_type: ai
spec:
  name: times2-plus2-raw
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: main
          image: gcr.io/science-experiments-divya/plus2:raw-0.1
          imagePullPolicy: Always
        - name: preprocess
          image: gcr.io/science-experiments-divya/times2:raw-preprocess-0.1
        imagePullSecrets: 
          - name: gcr-json-key
    graph:
      name: preprocess
      type: TRANSFORMER
      endpoint:
        type: REST
      children: 
      - name: main
        type: MODEL
        endpoint:
          type: REST
    name: raw-example
    replicas: 1

```

# Attempt to use predict-raw
- Changes only the model

Input:
`{"jsonData":{"X":10},"meta":{}}`


# Version History
- 0.1: Base code
- 0.1.1: Input-output, changes all methods to raw

