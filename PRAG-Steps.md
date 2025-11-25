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
# 5. Data Augmentation

This is the process of creating variations of my original questions for the dataset using the retrieved documents, to train more more parametric representations:

```bash
python .\src\augment.py --model_name Qwen/Qwen2.5-1.5B-Instruct2 --dataset scierc --data_path ./data/scierc/questions.json --sample 5 --topk 3
```

# 6. Data Encoding

This will create a parameterized representation of the documents (LoRa) for the given dataset.

The output of the last phase will be under: ```F:\Universidade\CL\Git-proj\PRAG\data_aug\scierc\Qwen\Qwen2.5-1.5B-Instruct``` for the inference the file must be named ```total.json```

```bash
python .\src\encode.py --model_name Qwen/Qwen2.5-1.5B-Instruct --dataset scierc --data_type questions --per_device_train_batch_size 1 --num_train_epochs 2 --num_train_epochs 2 --learning_rate 1e-4 --lora_rank 8 --lora_alpha 16 --sample 10
```

