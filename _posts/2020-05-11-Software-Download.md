---
published: true
---
Amongst all the problems of having new things to understand that came in parcel of a new internship is the downloading of softwares.


# Jupter notebook

## Installation

1. `pip install jupyterlab`

2. Check it exist: `jupyter notebook --version`


## Running
Start server: `jupyter lab`

## Problems

1. Unable to run 
```
UPDATE w/ FIX: I fixed this problem by adding *anaconda root*/Library/bin to my PATH.

I am also having this issue with version 2018.12

$ Traceback (most recent call last):
File "H:\Anaconda3\lib\site-packages\notebook\services\sessions\sessionmanager.py", line 10, in import sqlite3
File "H:\Anaconda3\lib\sqlite3_init_.py", line 23, in
from sqlite3.dbapi2 import *
File "H:\Anaconda3\lib\sqlite3\dbapi2.py", line 27, in
from _sqlite3 import *
ImportError: DLL load failed: The specified module could not be found. During handling of the above exception, another exception occurred: Traceback (most recent call last):
File "H:\Anaconda3\Scripts\jupyter-notebook-script.py", line 6, in
from notebook.notebookapp import main
File "H:\Anaconda3\lib\site-packages\notebook\notebookapp.py", line 86, in
from .services.sessions.sessionmanager import SessionManager
File "H:\Anaconda3\lib\site-packages\notebook\services\sessions\sessionmanager.py", line 13, in from pysqlite2 import dbapi2 as sqlite3 ModuleNotFoundError: No module named 'pysqlite2'
```
Fixes: Include path into system variables



