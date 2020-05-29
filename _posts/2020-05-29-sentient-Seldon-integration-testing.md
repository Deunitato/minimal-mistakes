---
published: false
---
Today I did some testing for seldon-core

### Things covered
- Metrics endpoint 
- Metadata (Change of structure)
- Seldon-core testing / seldon-core testing framework


# Metadata 

Sample code:
```
import logging
import base64
class simple(object):
    """
    Model template. You can load your model parameters in __init__ from a location accessible at runtime
    """

    def __init__(self):
        """
        Add any initialization parameters. These will be passed at runtime from the graph definition parameters defined in your seldondeployment kubernetes resource manifest.
        """
        print("Initializing")

        return

    def health_status(self):
    #response = self.predict([1, 2], ["f1", "f2"])
    #assert len(response) == 2, "health check returning bad predictions" # or some other simple validation
        return "I am super healthy :)"

    def metadata(self):

        meta = {
            "name": "model-name",
            "versions": ["model-version"],
            "platform": "platform-name",
            "inputs": [{"name": "input", "datatype": "BYTES", "shape": [1]}],
            "outputs": [{"name": "output", "datatype": "BYTES", "shape": [1]}],
        }

        return meta

    def init_metadata(self):

        meta = {
            "name": "model-name",
            "versions": ["model-version"],
            "platform": "platform-name",
            "inputs": [{"name": "input", "datatype": "BYTES", "shape": [1]}],
            "outputs": [{"name": "output", "datatype": "BYTES", "shape": [1]}],
        }

        return meta

    def metrics(self):
        return [
            {"type": "COUNTER", "key": "mycounter", "value": 1}, # a counter which will increase by the given value
            {"type": "GAUGE", "key": "mygauge", "value": 100},   # a gauge which will be set to given value
            {"type": "TIMER", "key": "mytimer", "value": 20.2},  # a timer which will add sum and count metrics - assumed millisecs
        ]
        
    def predict(self, X, feature_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info(X)
        
        return X


```

Model_deployment.yaml:
```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-simple
  namespace: default
  labels:
    model_name: simple-model
    api_type: microservice
    microservice_type: ai
spec:
  name: times2-plus2
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: model
          image: gcr.io/science-experiments-divya/simple:input-output
          imagePullPolicy: Always
    graph:
      name: model
      type: MODEL
      endpoint:
        type: REST
    name: meta-metrics-test
    replicas: 1

```

Executed commands:

`docker build -t gcr.io/science-experiments-divya/simple:input-output .`

`docker push gcr.io/science-experiments-divya/simple:input-output`

`kubectl apply -f model_deployment.yaml`

### Workability
Prefix used:
`/seldon/default/seldon-simple/`

Called prediction with `{"jsonData": {"X" : 4 }}`

Response:
```json
{
    "jsonData": {
        "X": 4
    },
    "meta": {
        "metrics": [
            {
                "key": "mycounter",
                "type": "COUNTER",
                "value": 1
            },
            {
                "key": "mygauge",
                "type": "GAUGE",
                "value": 100
            },
            {
                "key": "mytimer",
                "type": "TIMER",
                "value": 20.2
            }
        ]
    }
}
```

## Metrics endpoint

Sending with `{"jsonData": {"X" : 4 }}`

404 page not found:
- `/api/v0.1/prometheus/metrics`
- `/api/v0.1/metrics`
- 	



References:
[Seldon-core Metrics code](https://docs.seldon.io/projects/seldon-core/en/latest/_modules/seldon_core/metrics.html), [Seldon-core wrapper source code](https://docs.seldon.io/projects/seldon-core/en/latest/_modules/seldon_core/wrapper.html?highlight=%2Fhealth),[Exposing prometheus /metric wrapper issue](https://github.com/SeldonIO/seldon-core/issues/1476),[Seldon-core metrics documentation](https://docs.seldon.io/projects/seldon-core/en/v1.1.0/analytics/analytics.html?highlight=metrics%20endpoint#metrics)
