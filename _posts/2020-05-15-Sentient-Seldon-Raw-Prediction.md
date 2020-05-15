---
published: true
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

## V0.1.1 (both)
- Successful sending of data

> Predefined json
![kube_raw_1.PNG]({{site.baseurl}}/img/kube_raw_1.PNG)


> Own Json Type
![kube_raw_2.PNG]({{site.baseurl}}/img/kube_raw_2.PNG)

> More than one values
![kube_raw_3.PNG]({{site.baseurl}}/img/kube_raw_3.PNG)

Log file:

Input: `{"data": {"X": 10}}`

Log output:
`INFO:  Char's log (model): {'data': {'X': 10}}`

## Model - V0.2

Updated predict_raw snippet:

Input: `{"data":{"X":10}}`

(0.2.1)
```
def predict_raw(self, X):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info("Char's log (model): " + str(X))
        
        data = X.get("data", {}).get("X")
        value = int(data) + 2
        return value
```

> There was an error with this

```
Traceback (most recent call last): File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 2447, in wsgi_app response = self.full_dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request rv = self.handle_user_exception(e) File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function return cors_after_request(app.make_response(f(*args, **kwargs))) File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception reraise(exc_type, exc_value, tb) File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise raise value File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 50, in Predict user_model, requestJson, seldon_metrics File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 84, in predict handle_raw_custom_metrics(response, seldon_metrics, is_proto) File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 48, in handle_raw_custom_metrics metrics = msg.get("meta", {}).get("metrics", []) AttributeError: 'int' object has no attribute 'get'
```

Returned server error: 500

Failed test input:
1. `{"data":{"X":10}, "meta":{"metrics":[]}}`

2. 
```
			{"data":{"X":10}, "meta":{"metrics":[
            {"type": "COUNTER", "key": "mycounter", "value": 1}, 
            {"type": "GAUGE", "key": "mygauge", "value": 100},   
            {"type": "TIMER", "key": "mytimer", "value": 20.2}]
            }}
```
3. `{"data":{"X":10}, "meta":{"metrics": {"type": 1}}}`
4. `{"data":{"X":10}, "meta":{"metrics": []}}`
5. `{"data":{"X":10}, "meta":{"metrics": "some metric"}}`
6. `{"data":{"X":10},"meta":{}}`


(0.2.2)
plus2 code snippet:

```
        x = json.loads(X)
        data = x.get("data").get("X")
        value = int(data) + 2
        return value
```
> Attempt to load it as a Json file instead

- Fails

```
line 2447, in wsgi_app response = self.full_dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request rv = self.handle_user_exception(e) File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function return cors_after_request(app.make_response(f(*args, **kwargs))) File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception reraise(exc_type, exc_value, tb) File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise raise value File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 50, in Predict user_model, requestJson, seldon_metrics File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 83, in predict response = user_model.predict_raw(request) File "/app/plus2.py", line 66, in predict_raw x = json.loads(X) File "/usr/local/lib/python3.7/json/__init__.py", line 341, in loads raise TypeError(f'the JSON object must be str, bytes or bytearray, ' TypeError: the JSON object must be str, bytes or bytearray, not dict
```

> Suspicion that X is a dict

(0.2.3)

- Added metric function

> No difference


Input:
`{"data":{"X":10},"meta":{"metrics":{"type": 1, "counter": 3}}}`

Error
```
Traceback (most recent call last): File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 2447, in wsgi_app response = self.full_dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1952, in full_dispatch_request rv = self.handle_user_exception(e) File "/usr/local/lib/python3.7/site-packages/flask_cors/extension.py", line 161, in wrapped_function return cors_after_request(app.make_response(f(*args, **kwargs))) File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1821, in handle_user_exception reraise(exc_type, exc_value, tb) File "/usr/local/lib/python3.7/site-packages/flask/_compat.py", line 39, in reraise raise value File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1950, in full_dispatch_request rv = self.dispatch_request() File "/usr/local/lib/python3.7/site-packages/flask/app.py", line 1936, in dispatch_request return self.view_functions[rule.endpoint](**req.view_args) File "/usr/local/lib/python3.7/site-packages/seldon_core/wrapper.py", line 78, in TransformInput user_model, requestJson, seldon_metrics File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 198, in transform_input handle_raw_custom_metrics(response, seldon_metrics, is_proto) File "/usr/local/lib/python3.7/site-packages/seldon_core/seldon_methods.py", line 51, in handle_raw_custom_metrics seldon_metrics.update(metrics) File "/usr/local/lib/python3.7/site-packages/seldon_core/metrics.py", line 79, in update metrics_type = metrics.get("type", "COUNTER") AttributeError: 'str' object has no attribute 'get'
```

(0.2.4)

Changed the return type to dictionary
![kube_raw_4.PNG]({{site.baseurl}}/img/kube_raw_4.PNG)

> Successful data sent



# Version History
- 0.1: Base code
- 0.1.1: Input-output, changes all methods to raw
- 0.2s: Modification of data in raw methods
