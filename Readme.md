# Document Structure

 - Introduction
 - Terminology
 - Dataset naming conventions
 - Artefact structure
 - Prerequisites
 - Environment Setup
 - Running Experiments
 

## Introduction


This replication package includes:

1. **A large dataset of KPIs** collected from **Alemira**, a commercial Learning Managing System developed in Constructor Tech and currently in use in several educaitonal insitutions. **Alemira** is a microservice-based application deploied on Kubernetes that takes full advantaget of its autoscaling mechanisms.
2. **The results of the experiments of PREFACE**, _PREdicting Failures in AutosCaling distributEd Applications_, the approach presented in our manuscript which predict and localize failures in autoscaling distributed applications.
3. **The toolset to execute PREFACE** to replicate the results obtained based on the provided dataset.

**PREFACE** combines descriptive statistics with a generative neural network (autoencoder) to reveal anomalous KPI values that are symptoms of incoming system failures, and ranks the microservices that are likely responsible for the failure. **PREFACE** introduces a prepocessing step exploiting descriptive statistics, to deal with time series of KPI sets with size that varies over time, as in autoscaling distributed applications.

## Terminology

* **KPI**: Key Performance Indicators, values of the metrics collected from the **Alemira** system on a microservice level.
* **Rectifier**: the component of PREFACE that reduces the variable input KPI series into a constant KPI set using descriptive statistics
* **Deep Autoencoder**: the component of PREFACE that identifies the anomalous KPIs by computing the reconstruction error for each KPI alongside the overall reconstruction error.
* **Localizer**: this component aggregates the score of the anomalous KPIs that belong to the same micorservice and ranks them, signaling as failing microserivces the top ranked at each timestamp for which PREFACE predicts an anomalous state.

## Dataset naming conventions

The dataset collected during normal execution is named as follow:
* `normal_1_14.csv`: this is the dataset that comprises the data collected over two weeks of normal, fault-free, execution and is used to train and validate the Deep Autoencoder.

The datasets collected during the execution with injected failures. The naming for these datasets follows the following convention: {failure-type}-{target service}-{unique identifier}. The failure type can be one of the following:
* `linear-cpu-stress-userapi-051516.csv`
* `linear-cpu-stress-redis-091514.csv`
* `linear-memory-stress-userapi-051218.csv`
* `linear-memory-stress-redis-091522.csv`
* `linear-network-delay-userapi-051816.csv`
* `linear-network-delay-redis-092016.csv`

## Artefact Structure
**TO WRITE**

Folders in output:
 - datasets
 	- Tuned
 	- Normalized
 - predictions
 - anomalies_lists_services_only
 - anomalies_lists_services_only_sliding_window
 - localisations_re_sliding_window

 - models -- save aoutoencoder model
 - other -- experiment timing
 - localisations_re -- not used
 - anomalies_lists -- used for debug
 - kpis_not_seen_in_prod -- utilitarial role
### Folders and files explanation:
**TO WRITE**


## Prerequisites


### Machine configuration for data analysis
- OS: MacOS Catalina
- Processor: 2.2 GHz 6-Core Intel Core i7
- Memory: 16 GB 2400 MHz DDR4
- Software packages
    - [Python 3](https://www.python.org/downloads/)
    - [Conda](https://docs.conda.io/projects/miniconda/en/latest/miniconda-install.html)

## Environment Setup


### Step 1. Create conda environment
```sh
conda create --name alemira-analysis --channel conda-forge python=3.10 jupyterlab numpy pandas scikit-learn matplotlib plotly scipy tensorflow
```

### Step 2. Activate conda environment
```sh
conda activate alemira-analysis
```

### Step 3. Open jupyter notebooks in the project folder
```sh
jupyter lab
```

### Step 4. Run following notebooks one by one
1. data_set_normalize.ipynb
2. dataset_tune.ipynb
3. predict.ipynb
4. results.ipynb

## Running Experiments
Running the experiments consists in multiple passages, namely: _preprocessing of the data_, _prediction generation_, and _results visualization_. Following, for each steps we indicated which scripts is to use and what are its inputs and outputs.
### Preprocess the data

1. Script: _dataset_tune.ipynb_

   - Purpose: data proprocessing
     - Input: _input/datasets/Consolidated_ - raw data with each data set consolidated in a single .csv file
     - Output: _output/output-111/datasets/Tuned_ - preprocessed data


2. Script: _data_set_normalize.ipynb_

   - Purpose: data normalization and smothing by average of 3 recent points
     - Input: _output/output-111/datasets/Tuned_ - preprocessed data
     - Output: _output/output-111/datasets/Normalized_ - normalized data 

### Predictions Generation

3. Script: _predict.ipynb_

   - Purpose: trains the model 
     - Input: _output/output-111/datasets/Normalized_ - normalized normal data 
     - Output: _model_ - trained autoencoder model 
   - Purpose: makes and vizualize timestamp level preditions
     - Input: _output/output-111/datasets/Normalized_ - normalized data with injected failures 
     - Output: _output/output-111/predictions_  
   - Purpose: detects KPI level anomalies
     - Input: _output/output-111/datasets/Normalized_ - normalized data with injected failures
     - Output: _output/output-111/anomalies_lists_services_only_
     - Output: _output/output-111/anomalies_lists_services_only_sliding_window_file_path_
   - Purpose: localize failures
     - Input: _output/output-111/anomalies_lists_services_only_sliding_window_file_path_ 
     - Output: _output/output-111/localisations_by_reconstruction_error_sliding_window_file_path_

### Results Visualization

4. Script: _results.ipynb_

   - Purpose: vizualization of the results
     - Input: _output/output-111/predictions_
     - Input: _output/output-111/localisations_re_sliding_window_
     - Output: _visualization_ 

