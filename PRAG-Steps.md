# 1. Prepare the Environment

Requirements should be installed from the requirements.txt within the PRAG tuto

```pip install -r requirements.txt```

# 2. Configuring the Root Directory

The file ```PRAG/src/root_dir_path.py``` should be edited:

```ROOT_DIR = "F:/Universidade/CL/Git-proj/PRAG"```

# 3. Preparing Data
## 3.1 Downloading Wiki knowledge base

Create directory "./PRAG/data/dpr"

Donwload through browser the wiki dump "https://dl.fbaipublicfiles.com/dpr/wikipedia_split/psgs_w100.tsv.gz" and extract it to the directory created

## 3.2 Configuring  ElasticSearch for retrieval

As this is on a windows some changes had to be done:
- Donwloaded ElasticSearch through a docker container: ```docker run -d --name elasticsearch -p 9200:9200 -p 9300:9300 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.15.0```
- In ```pre_elastic.py``` in the function ```generate_actions()``` it had to be included the ```enconding='utf-8'```
- Finally index Wikipedia data ```python prep_elastic.py --data_path data/dpr/psgs_w100.tsv --index_name wiki```
- To test you can use this commands:
    - ```curl -X GET "localhost:9200/"```
    - ```curl -X GET "localhost:9200/_cat/indices/wiki?v"```
    - ```curl -X GET "localhost:9200/wiki/_count"```

# 4. Preparing Dataset

A directory under the directory ```./PRAG/data/``` should be created for our dataset and in this we should have a json file with the QA pairs for the dataset in this format:

```json
[
    {
        "question": "string",
        "answer": "string or list[string]",
    }
]
```
# Data Augmentation


