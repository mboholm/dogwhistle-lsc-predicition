# Code for paper "Can political dogwhistles be predicted by distributional methods for analysis of lexical semantic change?"

This repository contains the code for the paper "Can political dogwhistles be predicted by distributional methods for analysis of lexical semantic change?".

## Corpus data
The original data sets are hosted by  [Språkbanken](https://spraakbanken.gu.se/): 
* *[Flashback Politik](https://spraakbanken.gu.se/lb/resurser/meningsmangder/flashback-politik.xml.bz2)*



## XML to training data
The XML files have, after decompression, been processed by `data/read_large_xml.py`, which extracts only the text of each example (without tags) and save it to a text file corresponding to its year (one example per line). 

Example:
```
python read_large_xml.py INPUT.xml PROTO_CORPUS
```

## Prepocessing for SGNS
Data from Step 1 were further preprocessed through `data/preprocess2.py`, in different ways for SGNS and LLMS. 

### 1. lower case, remove emojis, remove numbers, remove punctuations, remove urls
For the SGNS model, data are preprocessed with configuration file `data/utils/pp_sgns_config.json`. Example:

```
python preprocess2.py PROTO_CORPUS SGNS_CORPUS1 --config utils/pp_sgns_config.json
```

### 2. Lemmatization and splitting of compounds of DWEs
DWEs are lemmatized and compound DWEs are split. Code in Jupyter Notebook: `data/preprocess_dwt.ipynb` (input: `SGNS_CORPUS1`, output: `SGNS_CORPUS2`). This process uses file for inflectional rules: `data/utils/dwts.paradigm`. 

## Prepocessing for LLMs
### 1. lower case, remove emojis, remove urls 
For the LLMs models, data are preprocessed with configuration file `data/utils/pp_bert_config.json`. Example:

```
python preprocess2.py PROTO_CORPUS LLM_CORPUS --config utils/pp_bert_config.json
```

## Collect DWE examples for LLMs
Collect examples of DWEs with `data/context_collector.py`; uses `data/utils/dwts.paradigm` to pair DWEs with their contexts. Example:

```
python context_collector.py LLM_CORPUS CONTEXTS utils/dwts.paradigm
```

**Structure of `CONTEXTS`:** One file per year, where each line is a tab separated tripple of

```
DWE N   EXAMPLE
```
where N is the number of DWEs in the example. 

## From text to vectors with LLMs
For LLM models sentences were vectorized by (input: `CONTEXTS`; output: `VECTORS`):

* `LLMs/sbert.ipynb`
* `LLMs/bert.ipynb`
* `LLMs/mt5.ipynb`

## Additional features
Modeling and results uses two additional features:

**Vocabulary and word counts:** 
Training of SGNS models and creation of results dataframe requires vocaublary files. These are built through `data/word_counter.py`. Example, with minimum frequency = 10:

```
python word_counter.py SGNS_CORPUS2 SGNS_VOCAB --min_count 10
```

For data preprocessed for LLMs use parameter `--column` to specify column of the examples (see above). Example:

```
python word_counter.py SGNS_CORPUS2 SGNS_VOCAB --min_count 10 --column 2
```


**Token counts (number of words and examples per year):** 
Calculations of normalized frequencies (frequency per million, fpm) requires total counts of words per year. These are built through `data/tok_counter.py`. 

```
python tok_counter.py CORPUS/SGNS_CORPUS2 CORPUS/extok_counts.json
```

## Diachronic vectors with skipgram with negative sampleing (SGNS)
The general procedure for training SGNS models is:

* An input corpus of text files (`SGNS_CORPUS2`); a directory with text files such that every file correspond to a time bin (here: years) and every line of a file correspond to a sentence.
* Vocabularies of each time bin (here: years) (`SGNS_VOCAB`).
* A script to:
    * build word vectors from texts of every time bin (here: years)
    * compare vectors over time 
* An output directory of results: vectors of terms over time. 

Models are built through `sgns/sgns.py`, which implements `word2vec` from [Gensim](https://radimrehurek.com/gensim/). The procedure for training models with shuffled controls is implemented in a Bash script, `sgns/train_serial.sh`. Parameters of training are altered within the Bash script. A big thank you goes to [`semantic-shift-in-social-networks`](https://github.com/GU-CLASP/semantic-shift-in-social-networks) for code on which this code is built. 

## Diachronic vectors with the LLMs
The general procedure for representing DWEs over time is:

* An input corpus of vectors (`VECTORS`)
* A script to:
    * build average vectors of DWEs *each year*
    * compare vectors over time
* An output directory of results

For building average vectors of each time period as well as for shuffled controls, the procedure was implemented with `LLMs/LLM_change.ipynb`.

## Collect LSC data
From the results produced by SGNS, SBERT-AVG and SBERT-CLT, tables can be produced by `analysis/create_df_mb.py`, which assumes:
1. an input directory `corpus` containing: 
* a directory `files` for examples, 
* a directory `vocab` for word counts, and
* `extok.json` for token counts per year to calculate normailsed frequencies. 
2. a results directory `measures` that contains
* cosine change measures `cosine_change`, 
* cosine similarity measures `cosine_similarity`

For adding relative frequencies use `--rel_freq`; for only considering DWEs in SGNS models use `--restrict_words`

Examples:
```
python create_df_mb.py CORPUS RESULTS PATH_TO_CSV --rel_freq
```

```
python CORPUS RESULTS PATH_TO_CSV --rel_freq --restrict_words "../data/utils/dwts.txt"
```

## Replacement data
The original replacement data are organised by (a) dogwhistle expressions (DWEs), (b) in two waves. Each DWE--wave pair is a column in th original csv-file. `replacement_data/extractor.py` reorganises the data in the following format, used in the next step:

```
output_dir
    dog_whistle_expression1
        in_group
            first_round
                replacements.txt
            second_round
                replacements.txt
        out_group
            first_round
                replacements.txt
            second_round
                replacements.txt
    dog_whistle_expression2
        ...
```
## Text to vectors for replacments for LLMs
For the LLMs, replacemetns are vectorized by:

* For BERT, `replacement_data/build_BERT_repl_vectors.ipynb`
* For mT5, `replacement_data/build_mT5_repl_vectors.ipynb`
* For SBERT, `replacement_data/build_sbert_repl_vectors.ipynb`

## In-group and out-group scores
Data for in-group and out-group scores are collected by:

* For SGNS, `replacement_data/sgns_inoutgroup_dimensions.ipynb`
* For LLMs, `replacement_data/llm_inoutgroup_dimensions.ipynb`

## Regression analysis
Statistical analysis is found in `analysis/analysis.ipynb`. 


