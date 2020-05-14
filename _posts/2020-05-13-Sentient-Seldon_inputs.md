---
published: true
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

# Using jsonData

Input file:

`{"jsonData":{"X":4}}`


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

`{"jsonData":{"X":10},"meta":{}}`


> The output returns 6 because it actually goes through a preprocessor which multiply the data by 2 before the main model adds it by 2

As seen in the main code, `X` gives a dictionary because the `jsonData` input specifies a dictionary. I manage to use the `get("X")` function, manipulate it and add it back to an dictionary. 

### Returning more than one values


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

In the transform model:
```
    def transform_input(self, X, feature_names):
        logging.info(X)

        return "THis is a trial";

```

Output:
`{"meta":{},"strData":"THis is a trial"}`

> Works fine

### Try to append the string

In the transform model:
```
    def transform_input(self, X, feature_names):
        logging.info(X)
        mystr = X + "appended at times2"

        return mystr;

```

> Just return X for main model for now

- I appended the string "appended in times2" to x before returning it

### Attempting to parse in a string and return a dictionary

- Using the same input

Preprocess model:
```
def transform_input(self, X, feature_names):
        logging.info(X)
        mydict = {}
        mydict["X"] = 2
        
        return mydict
```

Output:

`{"jsonData":{"X":2},"meta":{}}`

> We can return it of different types, does not have to return the same type as what was returned
>
> It seems like seldon was able to detect what type it is and change the json file directly.

## binData

Input:
`{"binData":"b'hi'"}`

> Unsure how to represent bytestring in json, so we use the b symbol

preprocess code:
```
    def transform_input(self, X, feature_names):
        logging.info(X)
        
        #Code for strData
        myStrin = X.decode() + " appended byte string in times2"
        return myStrin
```

> This does not work unfortunately, due to the input. Json does not understand b.

Converting the string "Hello \n" to base64 yields SGVsbG8K, so i change the input to

`{"binData":"SGVsbG8K"}`

Output:

`{"meta":{},"strData":"Hello\n appended byte string in times2"}`

### Converting string to bindata

Using the same input, make changes to the main model.
> Note that the preprocess transform method outputs a string

```
import base64

...
...

def predict(self, X, feature_names):
        """
        Return a prediction.

        Parameters
        ----------
        X : array-like
        """
        print("Predict called - will run plus2 function")
        logging.info(X)
        encodedBytes = base64.b64encode(X.encode("utf-8"))
        return encodedBytes
```

Output:

`{"binData":"U0dWc2JHOEtJR0Z3Y0dWdVpHVmtJR0o1ZEdVZ2MzUnlhVzVuSUdsdUlIUnBiV1Z6TWc9PQ==","meta":{}}`

> Successful conversion of string to bindata
>
> Once again, seldon was able to detect the type of data being produce and store it accordingly to the json tag

# Trying new methods

```
const (
	UNKNOWN_TYPE       PredictiveUnitType = "UNKNOWN_TYPE"
	ROUTER             PredictiveUnitType = "ROUTER"
	COMBINER           PredictiveUnitType = "COMBINER"
	MODEL              PredictiveUnitType = "MODEL"
	TRANSFORMER        PredictiveUnitType = "TRANSFORMER"
	OUTPUT_TRANSFORMER PredictiveUnitType = "OUTPUT_TRANSFORMER"
)
```

## Transport_Output


### Failed attempts
- Tried to use both input and output in the same file, doesnt work.
- Tried to only have output as its own file. Doesnt work

> Found out that yaml file output must be change
- Works

Deployment.yaml's graph snippet:
```
    graph:
      name: preprocess
      type: OUTPUT_TRANSFORMER
      endpoint:
        type: REST
      children: 
      - name: model
        type: MODEL
        endpoint:
          type: REST
    name: times2-plus2-pod
    replicas: 1
```

Preprocess:

```
def transform_output(self, X, feature_names):
        logging.info(X)
        return "It has transformed, the old was " + X

```

# Combiners

Good example of Combiner: https://github.com/Deunitato-sentient/seldon-core/tree/master/examples/combiners/spam_clf_combiner

Following the example, I will attempt to create a program that makes use of combiners.

Containers:
- 2 sub Models
- 1 Combiner

Structure:
```
subModel1 ->  ---------
			  |Combiner|
subModel2 ->   --------
```

subModel1 -> Attempt to + 2
subModel2 -> Attempt to *2

Input:
`{"data": {"names": [],"ndarray": [[5]]}}`
- Only this works
- Tried JsonData but failed

Graph for yaml deploy file:
```
graph:
        name: combiner
        type: COMBINER
        endpoint:
          type: REST
        children: 
        - name: subprocess-times2
          type: MODEL
          endpoint:
            type: REST
        - name: subprocess-plus2
          type: MODEL
          endpoint:
            type: REST
    name: times2-plus2-pod
    replicas: 1
```

combiner.py:

```
class combiner(object):

    def aggregate(self, Xs, features_names=None):
        """average out the probabilities from multiple classifier and return that as a result"""
        # In this case we return the larger
        logging.info(Xs)
        if (Xs[1] > Xs[0] ):
            larger = Xs[1]
        else:
            larger = Xs[0]
        
        return larger

```

> The docker file must change to type combiner



plus2.py:

```
class plus2(object):
  def predict(self, X, feature_names):
          """
          Return a prediction.

          Parameters
          ----------
          X : array-like
          """
          print("Predict called - will run plus2 function")
          logging.info(X)

          return X + 2
```

times2.py:
```
class times2(object):
    def predict(self, X, features_names):
        logging.info(X)
        return X*2
```
Output data:
`{"data":{"names":["t:0"],"ndarray":[[10]]},"meta":{}}`

## Changing of input
Input - Multiple data:

`{"data": {"names": [],"ndarray": [[5],[2]]}}`
Using this will not work

Output:
`{"data": {"names": [],"ndarray": [[5],[2]]}}`


Changing the code in combiner.py:

```
def aggregate(self, Xs, features_names=None):
        """average out the probabilities from multiple classifier and return that as a result"""
        # In this case we return the larger
        logging.info("Char's Logging (Aggregate):"+ str(Xs))
        #Assuming that xs is an array of arrays
        count = 0
        model1 = Xs[0].tolist()
        model2 = Xs[1].tolist()
        listoflargeNumber = []

        for x in model1:
            if(x > model2[count]):
                listoflargeNumber.append(x)
            elif (x<model2[count]):
                listoflargeNumber.append(model2[count])
            else: #they are both equal
                listoflargeNumber.append(0)
            count = count + 1

        
        return listoflargeNumber
```

> Testing the data we realised that the combiner extracts it into a fix number of 2 arrays which can be access via using Xs[0] and Xs[1]

Depending on the input, these two array can hold a list of values or just one value.

In our second example of multiple input, we have `{"data": {"names": [],"ndarray": [[5],[2],[3]]}}` as the input.

Output:

`{"data":{"names":[],"ndarray":[[10],0,[6]]},"meta":{}}`

### jsonData

- This works the same with JsonData


Input:`{"jsonData":{"X":5, "Y": 6}}`

Aggregate code snippet

```
        model1 = Xs[0]
        model2 = Xs[1]
        mydict = {}
        assert (len(model1) == len(model2), "Dictionary size must be the same" )

        for (k,v), (k2,v2) in zip(model1.items(), model2.items()):
            if(v > v2):
                mydict[k] = v
            elif (v < v2):
                mydict[k2] = v2
            else: #they are both equal
                mydict[k] = 0
        
        return mydict
```

#### Notes
- Aggregate seems to work only for json type data


