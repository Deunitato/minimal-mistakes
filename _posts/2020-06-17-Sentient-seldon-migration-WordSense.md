---
published: true
---
# Things done today
- WordSense Migration

# Word Sense migration

## File Changes

> For the seldon_deployment file, the containername ,class name , file name everything should be same

### bert_util

1. Added model_dir in global
2. Change line 185: `args.dict['bert_model_path'] = model_dir + "/bert_model"`

### WordSenseDisambiguation

1. Shift Model function to the top
2. Shift UserClassException to the top
3. Added model_dir in wordsensedisambiguation class global
4. Change line 325 - 326:
	```python
        saver = tf.train.import_meta_graph( self.model_dir + "/saved_model/model.ckpt.meta")
        saver.restore(self.sess, self.model_dir + "/saved_model/model.ckpt")
    ```
5. Change self calling methods to include `self.` starter
	- e.g `return {"data":self.infer(text, target_word, repeat)}`
6. Remove `definition2tokenized_definition = {}` in line 180
7. Added `WordSenseDisambiguation.` header in global class methods
	- e.g line 317: `self.bert_util = BERT_UTIL(WordSenseDisambiguation.args,)`
8. added `import flask` at the imports

### Arg
1. Add model_dir
2. Add model_dir to paths line 12-18

### Dockerfile

1. Added run lines
```Dockerfile
RUN apt-get update
RUN apt-get -y upgrade
RUN apt-get install -y build-essential python3-dev python3-pip libopenblas-dev
RUN pip3 install -r requirements.txt
RUN python3 -m spacy download en_core_web_sm
RUN python3 -m nltk.downloader wordnet

```