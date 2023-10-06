# Document Structure

 - Introduction
 - Terminology
 - Dataset naming conventions
 - Replication Package structure
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

## Replication Package structure

This replication package is composed as follow:

* _dataset_tune.ipynb_ is the notebook responsible to tune the datasets, including alligning the failure injection dataset to the training dataset, removing columns that are constant, and columns with empty values.


* _data_set_normalize.ipynb_ is the notebook that normalizes the datasets using the _min-max_ normalization technique.


* _predict.ipynb_ is responsible to generate train the Autoencoder model and generate the predictions according to its reconstruction error. This notebook also calculate the ranking of the services in order to allow the localization of the failure


* _results.ipynb_ is used to generate the graphs and plots shown in the manuscript


* _input_ folder: this folder contains the dataset collected and need to run the experiments. More specifically:
  * _datasets_ contains the subfolder _Consolidated_ where all the datasets related to both the normal execution and the failure injection execution can be found.
  * _other_ contains the _failure-injection-log.csv_, where the information of each failure injection are stored, including _Failure Type_, _Failure Pattern_, _Target Service_, _Beginning of the Experiment_, _End of the Experiment_, _Name of the Relative Dataset_, and _System Disruption Timestamp_.


* _output_ folder: this folder contains all the output files generated from the scripts used. This file are saved in multiple subfolder contained in _output-111_. More specifically:
  * _datasets_ contains two subfolders, _Tuned_ and _Normalized_. These contain the preprocessed datasets and the normalized dataset according to the _min-max_ normalization technique respectively.
  * _predictions_ contains a .csv file for each failure injection dataset in which, for each timestamp, it stores a boolean value _1_ or _0_ indicating whether PREFACE predicted a failure or not.
  * _anomalies_list_ contains a .csv file for each failure injection dataset, were we stored the reconstruction error of each KPI for each timestamp. This is used for debugging purposes.
  * _anomalies_lists_services_only_ similarly, contains a .csv file for each failure injection dataset, were we stored the reconstruction error of each KPI related to the services ranked from the biggest to the smallest. This is used from the Localizer for the localization of the failure
  * _anomalies_lists_services_only_sliding_window_ as before, contains a .csv file for each failure injection dataset, were for each minute we stored the ranked z-score calculated using the sliding window method described in the manuscript.
  * _localisations_re_sliding_window_ includes a .csv file for each failure injection dataset, were we stored the ranking of the services using the z-score calculated as described in the manuscript.
  * _models_ is the folder in which we store the trained Autoencoder.
  * _other_ stores a .csv file detailing the timing of each failure injection, including the _Failure Injection Experiment Name_, the _Total Number of Timestamps_, the _Timestamp at which Failure Injection Started_ and the _Timestamp at which Failure Injection Ended_
  * kpis_not_seen_in_prod


* _predict_notebook_sections_ folder contains some Jupyter Notebooks with functions that are used from the four main scripts described before.


* _functions.ipynb_ is a notebook containing additional useful functions used by the scripts described before.

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

