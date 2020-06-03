---
published: true
---
# Things done today
- Seldon migration : Taxonomy
- Alfred testing


# Seldon migration
- Current idea is to rid the need for the flask and to use seldon framework instead


Changes:
- Change deploy_requirements.txt name to `requirement.txt`
- Add in seldon base methods
- Add in customUserexception class
- Copy all helper functions
- Replace seldon's init with the init found in the main file
- Set up environement if any (Refer to dockerfile)
- Ensure the predict_raw and metadata return the correct output format (dict)

## Base code

Base code:
```python
import os
import json
import logging
import re
import numpy as np
import pandas as pd
from collections import Counter
import networkx as nx
from networkx.algorithms import all_shortest_paths
import spacy
from flask import Flask, jsonify

# ======================================================================================================================
# Helper Functions
# ======================================================================================================================
def generate_nodes_from_category(category):
    
    all_nodes = []
        
    nodes = category.split("/")
    
    n_nodes = len(nodes)

    for i, j in enumerate(nodes):

        n_ = ("_".join([j.lower(), str(i)]), {"name": j, "level": i})
        
        if i == 0:
            
            n_[1].update({"type": "root"})
            
        elif i == (n_nodes - 1):
            
            n_[1].update({"type": "leaf"})
        
        else:
            
            n_[1].update({"type": "body"})

        all_nodes.append(n_)
    
    return all_nodes

def generate_category_nodes_edges(category):
    
    category = category.strip()
    
    try:
        
        assert (category is not None) or (len(category) > 0)
        
    except AssertionError:
        
        raise ValueError("Must input valid category.")
    
    else:
                
        cat_nodes = generate_nodes_from_category(category)

        cat_node_names = [i for i, j in cat_nodes]
        
        starting_nodes = cat_node_names[:-1]
        ending_nodes = cat_node_names[1:]
        
        return cat_nodes, list(zip(starting_nodes, ending_nodes))

def generate_graph(categories):
    
    all_nodes = []
    all_edges = []
    
    for cat in categories:
        
        n_, e_ = generate_category_nodes_edges(cat)
        
        all_nodes += n_
        all_edges += e_
        
    G = nx.DiGraph()
    
    G.add_nodes_from(all_nodes)
    G.add_edges_from(all_edges)
                
    return G

def get_graph_data(G, data_name, sort=True):
    
    data = pd.Series(dict(G.nodes.data(data_name)), name=data_name)
    
    return data.sort_values() if sort else data

def convert_node_ids_to_names(node_ids, node_name_map):

    return [node_name_map.loc[_] for _ in node_ids]


class Taxonomy(object):
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
    #tx = Taxonomy(taxonomy_json_path, embedding_path=glove_dir)

    # ==============================================================================
    # Seldon-core python wrapper methods
    # ==============================================================================
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
    
    def __init__(self, data_path = taxonomy_json_path, embedding_path = glove_dir, insist_dag=True, default_threshold=0.95):
        """Taxonomy (Generic)
        Finds the category a particular word belongs to given a pre-built taxonomy tree.
        
        The full taxonomy consists of a few major categories (for e.g. retail, cars, etc.). Each category is stored as a separate taxonomy tree within the `Taxonomy` instance.
        
        Each node in a taxonomy tree is identified using a node_id. It has attributes: name (a human readable string), level (how far it is from the root node) and type (whether it is a root node, leaf node or body node).
        
        Given a word, the `Taxonomy` object wil then try to match the word with the name of the leaf nodes. The entire branch from the matched leaf node to the root of the corresponding taxonomy tree will then be returned.
        
        Parameters
        ----------
        data_path : str or path-like object
            Path to taxonomy tree data (should be a JSON file)
        
        embedding_path : str or path-like object
            Path to directory containing word embedding vectors.
            
            Default: "./data/glove_vec"
            
        insist_dag : boolean
            Insist that taxonomy tree is a directed acyclical graph
        
        default_threshold : float
            Between 0 and 1. Minimum word similarity required when comparing match term with leaf nodes of taxonomy tree.
            
        Attributes
        ----------
        nlp : spacy language model
            Stores word embeddings
        
        graphs : dict of networkx graphs
            Stores taxonomy trees. Consists of various categories.
            
            Default: empty dict
        
        node_names : dict of pandas Series
            Mapping of taxonomy tree name (keys) to a pandas Series that maps the node ID to human-readable node names within that taxonomy tree
        
        node_levels : dict of pandas Series
            Mapping of taxonomy tree name (keys) to a pandas Series that maps the node ID to the node level within that taxonomy tree
        
        node_name_tokens : dict of pandas Series
            Mapping of taxonomy tree name (keys) to a pandas Series that maps the node ID to tokenised human-readable node names within that taxonomy tree. This is to facilitate comparison with match term using the spacy word embeddings later.
        
        node_data_names : list of str 
            List of node attributes.
            
            Default : ["level", "type", "name"]
            
        logger : logger instance
            Logger used to log processes.
        """
        
        self.data_path = data_path
        self.nlp = spacy.load(embedding_path)
        self.insist_dag = insist_dag
        self.default_threshold = default_threshold
        self.graphs = {}
        self.node_names = {}
        self.node_levels = {}
        self.node_name_tokens = {}
        self.node_data_names = ["level", "type", "name"]
        self.logger = logging.getLogger(__name__)
        
        # Initialise the model
        initialise()
        return

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
            "metadata": metadata
        }

    def predict_raw(self, request):

        # if request.method == "GET":

        #     return jsonify(metadata)
        text = request.get("text", None)
        threshold = request.get("threshold", 1)
        logger.info(text)

        if (text is None) or (len(text) == 0):
            raise UserCustomException('Bad Request. No data found.',403)

        elif len(text) > max_search_terms:
            raise UserCustomException("Too many search term requested. Max: 20. Received: {}".format(len(text)),403)

        else:
            return {
                "data": predictModel(text, threshold)
                }


    # ==============================================================================
    # Model methods
    # ==============================================================================
    def __load_data(self):
        
        assert os.path.isfile(self.data_path), "data file not found!"
        assert self.data_path[-4:].lower() == "json", "file should be a json"
        
        with open(self.data_path, "r") as f:
    
            raw_data = json.load(f)
        
        return raw_data
    
    def __check_dag(self):
        
        all_graphs_dag = True

        for i in self.graphs:

            is_dag = nx.is_directed_acyclic_graph(self.graphs[i])

            all_graphs_dag *= is_dag
            
            if is_dag:
                
                self.logger.debug("{} is DAG".format(i))
            
            else:
                
                self.logger.warn("{} is not DAG".format(i))

        if all_graphs_dag:
            
            self.logger.info("All graphs are DAG!")
            
        else:
            
            self.logger.warn("Some graphs not DAG!")
            
            if self.insist_dag:
                
                raise ValueError("All graphs must be DAG!")
            
    def initialise(self):
        """Initialise the taxonomy graphs
        """
        for i, j in self.__load_data().items():
            
            self.graphs[i] = generate_graph(j)
            
        self.__check_dag()
        
        for i in self.graphs:
            
            self.node_names[i] = self.get_graph_node_data(i, "name")
            self.node_levels[i] = self.get_graph_node_data(i, "level")
            self.node_name_tokens[i] = self.node_names[i].apply(lambda x: self.nlp(x.lower()))
            
        return
    
    def get_graph_node_data(self, graph_name, data_name, **kwargs):
        
        graph_name = graph_name.lower()
        data_name = data_name.lower()
        
        assert data_name in self.node_data_names, "data_name not found. Expect {}. Got {}."\
            .format(", ".join(self.node_data_names), data_name)
        
        assert graph_name in self.graphs, "Graph not found. Expect {}. Got {}."\
            .format(", ".join(list(self.graphs.keys())), graph_name)
        
        return get_graph_data(self.graphs[graph_name.lower()], data_name, **kwargs)
    
    def get_all_shortest_paths(self, graph_name, leaf, include_leaf=True):
        
        assert graph_name in self.graphs, "Graph not found. Expect {}. Got {}."\
            .format(", ".join(list(self.graphs.keys())), graph_name)
        
        paths = all_shortest_paths(self.graphs[graph_name], graph_name+"_0", leaf)
        
        if include_leaf:
            
            return ["/".join(convert_node_ids_to_names(_, self.node_names[graph_name])) for _ in paths]
        
        else:
            
            return ["/".join(convert_node_ids_to_names(_[:-1], self.node_names[graph_name])) for _ in paths]
        
    def match_graph_nodes(self, graph_name, match_term, threshold=1.):
        
        assert graph_name in self.graphs, "Graph not found. Expect {}. Got {}."\
            .format(", ".join(list(self.graphs.keys())), graph_name)
        
        doc = self.nlp(match_term)
        
        sims = self.node_name_tokens[graph_name]\
            .apply(lambda x: x.similarity(doc) if (np.sum(x.vector) > 0) and (np.sum(doc.vector) > 0) else 0)
        
        sims = sims.rename("similarity")
        
        node_sim = pd.concat([self.node_names[graph_name], sims], axis=1)
        
        return node_sim.loc[sims >= threshold]
    
    def get_categories(self, match_term, threshold=None, **kwargs):
        """Get the category of a particular match term
        
        Parameters
        ----------
        match_term : str
            Word to match on
        
        threshold : float
            Between 0 and 1. Minimum word similarity required when comparing match term with leaf nodes of taxonomy tree. 
            
            Default: None. If None, then use self.default_threshold
            
        kwargs 
            Optional arguments to be passed to `get_all_shortest_paths`
            
        Returns
        -------
        list of category strings
        """
        
        categories = []
        
        if threshold is None:
            
            threshold = self.default_threshold
        
        for i in self.graphs:
            
            self.logger.debug("Processing graph {}".format(i))
            
            node_sims = self.match_graph_nodes(i, match_term, threshold=threshold)
            
            for node_matched, node_sim in node_sims["similarity"].iteritems():
                
                categories += [(_, np.round(node_sim, 2)) for _ in self.get_all_shortest_paths(i, node_matched, **kwargs)]
                
        return categories
    
    def predictModel(self, all_match_terms, threshold=None):
        """Match categories for all terms
        
        Parameters
        ----------
        all_match_terms : list of str

        Returns
        -------
        dict of lists
            key: term being matched
            value: list containing tuples of `(category, similarity_score)`
        """
        
        result = {}
        
        for term in all_match_terms:
        
            result[term] = self.get_categories(term, threshold=threshold)
            
        return result

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


### Code base: Edit 1
> Included the class taxonomy functions and its helper function

Repo: <[Repo link](https://github.com/Deunitato-sentient/microservice-ai-taxonomy_generic/blob/initial_branch/deploy/TaxonomyGeneric.py)>



# Alfred


## Installation (Without Jupyter)
- Navigate to Alfred directory
- Run command `pip3 install -e ./Alfred`

Copy files

Home dir:
- config-engg.json
- config.yaml

Deploy file:
- Alfred_script.py


## Files

### Config-eng

- ambassador_mappingname: Follow the format `<model_name>_mappings`
- ambassador_customurl: Format is `/microservices/nlp/<modelTitle>/v0.1/getpredictions`
- ambassador_service: `<meta_data_name>/<graph_name>.default:8000`
- model_storage: The name of your container registry

e.g
```
    "ambassador_mappingname": "inverse_norm_mapping",
    "ambassador_customurl": "/microservices/nlp/inverseNorm/v0.1/getpredictions",
    "ambassador_service":"inverse-norm-pod.default:8000",
    "ambassador_timeout":60000
```


```
"model_storage": {"uri":"gs://science-experiments-divya"}}
```

Full sample:

```json
{
    "annotations":
    {
     "project_name":"inverseNorm",
     "deployment_version": "v1",
     "seldon_rest_timeout": "'60000'",
    "seldon_rest_connection_timeout": "'60000'",
    "seldon_grpc_read_timeout": "'60000'",
    "ambassador_mappingname": "inverse_norm_mapping",
    "ambassador_customurl": "/microservices/nlp/inverseNorm/v0.1/getpredictions",
    "ambassador_service":"inverse-norm-pod.default:8000",
    "ambassador_timeout":60000
	
    },
    
"livenessProbe": {
    "failureThreshold": 3,
    "initialDelaySeconds": 100,
    "periodSeconds": 5,
    "successThreshold": 1,
    "timeoutSeconds": 1
  },
  "readinessProbe": {
    "failureThreshold": 3,
    "initialDelaySeconds": 100,
    "periodSeconds": 5,
    "successThreshold": 1,
    "timeoutSeconds": 1
  },
"model_storage": {"uri":"gs://science-experiments-divya"}}
```

> Note that of all the values, the `ambassador_service` must follow the format of the yaml file, for the other config is custom

#### Extras:

Setting up environment
- We can use this code prepare an environemnt

```json
"env": [
    {
      "name": "GUNICORN",
      "value": "4"
    },
    {
      "name": "PORT",
      "value": "5000"
    }
  ],
```

### Config.yaml

Main important changes:
- metadataname
- graphname

> this is important as it affect the other file (json)

Sample:
```yaml
metadata_name : inverse
model_name : inverse_norm
models_used : "yes"
containers :
    - container_name : inverse
      tag_num : 0.1.0
      artefacts : "yes"
      docker_details:
        version : 0.1.0
        ENV:
          - MODEL_NAME : inverse_norm
            SERVICE_TYPE : MODEL
   
graph:
  name: inverse-norm
  type: MODEL
  endpoint:
    type: REST
  children: []
predictor_name: norm-pod
```

### Alfred_script.py

Changes:
- config yaml path

```python
# Saving the path of input config file filled by scientists
config_yaml="/mnt/c/Users/Charlotte/Desktop/NUS DOC/Code/sentient/microservice-ai-inverse_norm/config.yaml"

# Reading the input config file filled by engineering team
# Need to be commented if config-engg.json is not required
with open("/mnt/c/Users/Charlotte/Desktop/NUS DOC/Code/sentient/microservice-ai-inverse_norm/config-engg.json") as f:
  default_dict = json.load(f)
```

- Change the path to match the location

## Running it

Run the command python3 Alfred_scipt.py

Expected output:
- Dockerfile is generated
- Deployment yaml is generated

> Check through ensure its correct

