# A Unified MRC Framework for Named Entity Recognition 
The repository contains the code of the recent research advances in [Shannon.AI](http://www.shannonai.com). 

**A Unified MRC Framework for Named Entity Recognition** <br>
Xiaoya Li, Jingrong Feng, Yuxian Meng, Qinghong Han, Fei Wu and Jiwei Li<br>
In ACL 2020. [paper](https://arxiv.org/abs/1910.11476)<br>
If you find this repo helpful, please cite the following:
```latex
@article{li2019unified,
  title={A Unified MRC Framework for Named Entity Recognition},
  author={Li, Xiaoya and Feng, Jingrong and Meng, Yuxian and Han, Qinghong and Wu, Fei and Li, Jiwei},
  journal={arXiv preprint arXiv:1910.11476},
  year={2019}
}
```


## Overview 

The task of NER is normally divided into **nested** NER and **flat** NER depending on whether named entities are nested or not. Instead of treating the task of NER as a sequence labeling problem, we propose to formulate it as a SQuAD-style machine reading comprehension (MRC) task. <br>

For example, the task of assigning the [PER] label to *"[Washington] was born into slavery on the farm of James Burroughs"* is formalized as answering the question *"Which person is mentioned in the text?"*. <br>

By unifying flat and nested NER under an MRC framework, we're able to gain a huge improvement on both flat and nested NER datasets, which achives SOTA results.

We use `MRC-NER` to denote the proposed framework. <br>
Here are some of the **highlights**:

1. *MRC-NER* works better than BERT-Tagger with less training data. 
2. *MRC-NER* is capable of handling both flat and nested NER tasks under a unified framework.  
3. *MRC-NER* has a better zero-shot learning ability which can predicts labels unseen from the training set.  
4. The query encodes prior information about the entity category to extract and has the potential to disambiguate similar classes. 

## Experimental Results on Flat/Nested NER Datasets
Experiments are conducted on both *Flat* and *Nested* NER datasets. The proposed method achieves vast amount of performance boost over current SOTA models. <br>
We only list comparisons between our proposed method and previous SOTA in terms of span-level micro-averaged F1-score here. 
For more comparisons and span-level micro Precision/Recall scores, please check out our [paper](https://arxiv.org/abs/1910.11476.pdf). <br> 
### Flat NER Datasets
Evaluations are conducted on the widely-used bechmarks: `CoNLL2003`, `OntoNotes 5.0` for English; `MSRA`, `OntoNotes 4.0` for Chinese. We achieve new SOTA results on `OntoNotes 5.0`, `MSRA` and  `OntoNotes 4.0`, and comparable results on `CoNLL2003`.

| Dataset |  Eng-OntoNotes5.0 | Zh-MSRA | Zh-OntoNotes4.0 | 
|---|---|---|---|
| Previous SOTA | 89.16 | 95.54  | 81.63 | 
| Our method |  **91.11** | **95.75** | **82.11** | 
|  |  **(+1.95)** | **(+0.21)** | **(+0.48)** | 


### Nested NER Datasets
Evaluations are conducted on the widely-used `ACE 2004`, `ACE 2005`, `GENIA`, `KBP-2017` English datasets.

| Dataset | ACE 2004 | ACE 2005 | GENIA | KBP-2017 | 
|---|---|---|---|---|
| Previous SOTA | 84.7 | 84.33 | 78.31  | 74.60 | 
| Our method | **85.98** | **86.88** | **83.75** | **80.97** | 
|  | **(+1.28)** | **(+2.55)** | **(+5.44)** | **(+6.37)** | 

Previous SOTA:
 
* [DYGIE](https://www.aclweb.org/anthology/N19-1308/) for ACE 2004.
* [Seq2Seq-BERT](https://www.aclweb.org/anthology/P19-1527/) for ACE 2005 and GENIA.
* [ARN](https://www.aclweb.org/anthology/P19-1511/) for KBP2017. 

## Dependencies 

* Experiments are conducted on a Ubuntu GPU server with Python 3.6.  <br> 
Run `pip3 install -r requirements.txt`to install packages dependencies.

* Download and unzip `BERT-Base, Uncased English` and `BERT-Base, Chinese` pretrained checkpoints. Then follow the [guideline](https://huggingface.co/transformers/v2.5.0/converting_tensorflow_models.html) from huggingface to convert TF checkpoints to PyTorch. 



## Data Preprocess 

Firstly you should transform tagging-style annoations to a set of MRC-style  `(Query, Context, Answer)` triples. Here we provide an example to show how these two steps work. We have given the queries in `python3 ./data_preprocess/dump_query2file.py` for you. Feel free to write down your own's queries.
MRC-Style datasets could be found [here](https://drive.google.com/file/d/1KHRSTL_jn5PxQqz4prQ1No2E2wWcuxOd/view?usp=sharing).

***Step 1: Query Generation***

Write down queries for entity labels in `./data_preprocess/dump_query2file.py` and run `python3 ./data_preprocess/dump_query2file.py` to dump queries to the folder `./data_preprocess/queries`. 

***Step 2: Transform tagger-style annotations to MRC-style triples*** 

Run `./data_preprocess/example/generate_data.py` to generate MRC-style data `data_preprocess/example/mrc-dev_ace05.json` and `data_preprocess/example/mrc-dev_msra.json` for ACE 2005(nested) and Chinese MSRA(flat), respectively. 


####  Nested NER 

We take ACE2005 as an example for *NESTED NER* to illustrate the process of data prepration.  

Source files for `ACE2005` contains a list of json in the format : 

```json
{
"context": "begala dr . palmisano , again , thanks for staying with us through the break .",
"label": {
    "PER": [
        "1;2",
        "1;4",
        "11;12"],
    "ORG": [
        "1,2"]
}
}
```
It assumes queries for ACE2005 should be found in `../data_preprocess/queries/en_ace05.json`. 
The path for the queries should be registered in dictionary `queries_for_dataset` of `../data_preprocess/query_map.py`. 

Run the following commands to get MRC-style data files.

```python3 
$ python3 
> from data_preprocess.generate_mrc_dataset import generate_query_ner_dataset
> source_file_path = "$PATH-TO-TAGGER-ACE05$/dev_ace05.json"
> target_file_path = "$PATH-TO-MRC-ACE05$/mrc-dev_ace05.json"
> entity_sign = "nested" #"nested" for nested-NER; "flat" for flat-NER.
> dataset_name = "en_ace2005" 
> query_sign = "default"
> generate_query_ner_dataset(source_file_path, target_file_path, entity_sign=entity_sign, dataset_name=dataset_name, query_sign=query_sign)
```

After that, `$PATH-TO-MRC-ACE05$/mrc-dev_ace05.json` contains a list of jsons: 

```json 
{
"context": "begala dr . palmisano , again , thanks for staying with us through the break .",
"end_position": [
    2,
    4,
    12
    ],
"entity_label": "PER",
"impossible": false,
"qas_id": "4.3",
"query": "3",
"span_position": [
    "1;2",
    "1;4",
    "11;12"],
"start_position": [
    1,
    1,
    11]
}
```

####  Flat NER 

Take Chinese MSRA as an example to illuatrate the process for FLAT NER. 

Source files are in CoNLL format and entities are annotated with BMES scheme : 

```
begala B-PER
dr M-PER
palmisano E-PER
, O
again O 
, O
thanks O
for O
staying O
with O
us O
through O
the O
break O
. O
```

Queries for Chinese MSRA should be found in `./data_preprocess/queries/zh_msra.json`. 
The path for the queries should be registered in dictionary `queries_for_dataset` of `./data_preprocess/query_map.py`. 

Run the following commands to get MRC-style datasets: 

```python3 
$ python3 
> from data_preprocess.generate_mrc_dataset import generate_query_ner_dataset
> source_file_path = "$PATH-TO-TAGGER-ZhMSRA$/dev_msra.bmes"
> target_file_path = "$PATH-TO-MRC-ZhMSRA$/mrc-dev_msra.json"
> entity_sign = "flat" #"nested" for nested-NER; "flat" for flat-NER.
> dataset_name = "zh_msra" 
> query_sign = "default"
> generate_query_ner_dataset(source_file_path, target_file_path, entity_sign=entity_sign, dataset_name=dataset_name, query_sign=query_sign)
```

After that, `$PATH-TO-MRC-ZhMSRA$/mrc-dev_msra.json` contains a list of jsons: 

```json 
{
"context": "begala dr . palmisano , again , thanks for staying with us through the break .",
"end_position": [2],
"entity_label": "PER",
"impossible": false,
"qas_id": "4.3",
"query": "3",
"span_position": [
    "1;2"],
"start_position": [1]
}
```


## Training BERT MRC-NER Model
./test.sh







