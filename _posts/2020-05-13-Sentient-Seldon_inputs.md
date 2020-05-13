---
published: false
---
Today I have given up on the "ambassador" flop and decided to work on trying out the different input/output of the seldon-core module. 
This post covers my findings for my experiments.

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
