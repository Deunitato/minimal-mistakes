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


## Steps
1. Ensure that venv is setup
2. Run the python file

  `python train_iris.py`
  > It might take awhile
3. 



# Pytest