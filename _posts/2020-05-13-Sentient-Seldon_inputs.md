---
published: false
---
Today I have given up on the "ambassador" flop and decided to work on trying out the different input/output of the seldon-core module. 
This post covers my findings for my experiments.

> Health/Status liveness probe yaml file
```
apiVersion: machinelearning.seldon.io/v1alpha2
kind: SeldonDeployment
metadata:
  name: seldon-model-preprocess
  labels:
    model_name: times2-plus2
    api_type: microservice
    microservice_type: ai
spec:
  name: times2-plus2
  predictors:
  - componentSpecs:
    - spec:
        containers:
        - name: model
          image: gcr.io/science-experiments-divya/plus2:0.4
          imagePullPolicy: Always
          livenessProbe:
            failureThreshold: 3
            initialDelaySeconds: 60
            periodSeconds: 5
            successThreshold: 1
            httpGet:
              path: /health/status
              port: http
              scheme: HTTP
            timeoutSeconds: 1
          readinessProbe:
            failureThreshold: 3
            initialDelaySeconds: 20
            periodSeconds: 5
            successThreshold: 1
            httpGet:
              path: /health/status
              port: http
              scheme: HTTP
            timeoutSeconds: 1
        - name: preprocess
          image: gcr.io/science-experiments-divya/times2:preprocess-0.2
        imagePullSecrets: 
          - name: gcr-json-key
    graph:
      name: preprocess
      type: TRANSFORMER
      endpoint:
        type: REST
      children: 
      - name: model
        type: MODEL
        subtype: MICROSERVICE
        endpoint:
          type: REST
    name: times2-plus2-pod
    replicas: 1

```

Covered:
- Different Json outputs/Inputs
- Json Readup
- Seldon methods tryout

# Different Json outputs/Inputs

1. Data as a dictionary

Input file:
```
{"jsonData":{"X":4}}
```

Code for model:
```
 def predict(self, X, feature_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info(X)
        value = X.get("X") + 2
        mydict = {}
        mydict["X"] = value

        return mydict
```
> This is the predict function as predefined by the seldon-core module. This function will run when /predict is used as the endpoint

Output:
```
{"jsonData":{"X":10},"meta":{}}
```

> The output returns 6 because it actually goes through a preprocessor which multiply the data by 2 before the main model adds it by 2

As seen in the main code, `X` gives a dictionary because the `jsonData` input specifies a dictionary. I manage to use the `get("X")` function, manipulate it and add it back to an dictionary. 

## Returning more than one values


> Attempting to change the return value


```
 def predict(self, X, feature_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info(X)
        value = X.get("X") + 2
        mydict = {} 
        mydict["X"] = value
        mydict["Hel"] = "hello"

        return mydict
```

Return output:
`{"jsonData":{"Hel":"hello","X":10},"meta":{}}`

## Using strData

Input data:
`{"strData":"Hello, I am a string!"}`

- Try to append the string

In the transform model:
```
    def transform_input(self, X, feature_names):
        logging.info(X)
        """
        # This is the code from "jsonData"
        
        value = X.get("X") * 2
        mydict = {}
        mydict["X"] = value
        

        return mydict
        """
        #Code for strData
        myStrin = X + " appended in times2"
        return myStrin;

```

> Just return X for main model for now

- I appended the string "appended in times2" to x before returning it

