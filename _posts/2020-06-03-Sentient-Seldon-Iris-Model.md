---
published: true
---
# Things covered in seldon

- Alfred testing
- Iris model
- Pytest

# Alfred testing
- Continue from yesterday

> Apparently there is some issue regarding my Alfred installation whereby the generated file is wrong
>
> The deployment yaml, under graph, the endpoint is in {} which is wrong


## Wrong installation command
- Suspected that the correct way to install is `pip3 install -e .`


References:

<[Reference](https://stackoverflow.com/questions/42609943/what-is-the-use-case-for-pip-install-e)>,


> Conclusion: This issue was unsolved, tested on another person's mac and it work fine. Could be a windows/WSL problem

# Iris Model
References: <[Documentation - API testing](https://docs.seldon.io/projects/seldon-core/en/v0.3.0/workflow/api-testing.html)>, <[Iris model github workbook](https://github.com/SeldonIO/seldon-core/blob/master/examples/models/sklearn_iris/sklearn_iris.ipynb)>

### Virtual environment setup

`sudo apt-get install python3.7-venv`

`source env/bin/activate`

### Dependencies
- sklearn
- seldon-core


```
pip install sklearn
sudo apt-get install python3.7-dev`
pip install seldon-core
```
### Files:
- dockerfile
- IrisClassifier.py
- makefile
- model_deployment.yaml
- requirements.txt
- train_iris.py


Changes:

Requirements.txt:

```txt
scikit-learn
scipy
seldon-core
```

dockerfile:

```dockerfile
FROM python:3.7-slim

COPY . /app

WORKDIR /app

RUN pip install -r requirements.txt

EXPOSE 5000

ENV MODEL_NAME=IrisClassifier
ENV API_TYPE=REST
ENV SERVICE_TYPE=MODEL
ENV PERSISTENCE=0

CMD exec seldon-core-microservice $MODEL_NAME $API_TYPE \
    --service-type $SERVICE_TYPE \
    --persistence $PERSISTENCE 
```

## Steps
1. Ensure that venv is setup
2. Run the python file

  `python train_iris.py`
  > It might take awhile
  
  Expected output:
  ```
  Loading iris data set...
  Dataset loaded!
  Training model...
  Model trained!
  Saving model in IrisClassifier.sav
  Model saved!
  ```
3.  Build the docker image

`docker build -t gcr.io/science-experiments-divya/iris_example:0.1 .`


## Testing it

`docker run --rm --name mymodel -p 5000:5000 gcr.io/science-experiments-divya/iris_example:0.1`

Testing using Url:


`curl  -s http://localhost:5000/predict -H "Content-Type: application/json" -d '{"data":{"ndarray":[[5.964,4.006,2.081,1.031]]}}'`


Response:

`
{"data":{"names":["t:0","t:1","t:2"],"ndarray":[[0.9548873249364059,0.04505474761562512,5.7927447968953825e-05]]},"meta":{}}
`


# Pytest