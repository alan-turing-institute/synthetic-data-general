# What is QUIPP

> These notes are a mixture of documentation from QUiPP, discussions with members of the team, reflections from Camila and [notes](https://hackmd.io/tpy3YfaARPWYLA7Rle_5yQ) written by Callum.

## Description

The QUiPP software is a pipeline for generating synthetic tabular data, that uses a variety of methods as implemented by several libraries. QUiPP also provides measures of privacy and utility of the resulting data in a variety of contexts. Whilst the synthetic data generation methods come from external libraries, the privacy and utility metrics have been chosen and implemented by the QUiPP team. 

QUiPP is a makefile-based reproducible pipeline. The pipeline is highly configurable with input json files, which tells the pipeline which dataset to use, synthetic methods to run (as well as providing configuration intrinsic to each method), and then subsequently which privacy and utility metrics to calculate.

The input data is present as two files: a csv file which must contain column headings (along with the column data itself), and a json file describing the types of the columns used for synthesis.

The following figure indicates the full pipeline, as run on an input file called example.json. This input file has keywords *dataset* (the base of the filename to use for the original input data) and `synth-method` which refers to one of the synthesis methods. As output, the pipeline produces:

- A number of **output synthetic datasets** csv files (configurable by the input json file) e.g: synthetic_data_1.csv, synthetic_data_2.csv, ...
- **Privacy metrics** in json files as the disclosure risk privacy score (e.g disclosure_risk.json)
- **Utility metrics** in several json files such as correlation like utility metrics and classification tasks (e.g sklearn_classifiers.json)

![](https://i.imgur.com/rfSyvd1.png)

The file-base desing of the pipeline makes it very costumisable, allowing it run in any dataset (as long its format and variable types are properly described on the dataset json file) and any configuration of avalaible methods and assessment metrics. 

The kind of data format and types that can be synthesised do not depend on the QUiPP pipeline, but the synthesis method to use. 

Furthermore, this design makes the QUiPP extensible. For example, to add a new synthetic data method you just need a new folder with the method name, and a file named `run` (which is typically a python wrapper that picks up the values from the input json and passes them to a python script housing the method). 

Finally, QUiPP can also generate toy datasets that can be used in the pipeline.  This is not part of the main functionality of the pipieline, but can used for experimentation and was useful in early days of development of the software.      

## How to use

The installation instructions are found in the [README](https://github.com/alan-turing-institute/QUIPP-pipeline#local-installation) of the repo. 

In order to run, you need to understand the top-level directory structure which mirrors the data pipeline. Not all directories in there are relevant for running the pipeline, therefore I will only describe the ones that are (more detail description of the top level repo can be found in [here](https://github.com/alan-turing-institute/QUIPP-pipeline#top-level-directory-contents)).

Relevant directories for running QUiPP: 

 - `env-configuration`: Set-up of the computational environment needed
   by the pipeline and its dependencies

 - `generators`: Quickly generating toy input data for the pipeline from a
   few tunable and well-understood models.

 - `datasets`: Sample data that can be consumed by the pipeline.


 - `synth-methods`: One directory per library/tool, each of them
   implementing a complete synthesis method

 - `utility-metrics`: Scripts relating to computing the utility
   metrics

 - `privacy-metrics`: Scripts relating to computing the privacy
   metrics

 - `run-inputs`: Parameter json files (see below), one for each run


### Running the pipeline

1. Have an input json file, in `run-inputs/`, for each desired synthesis (a dataset + a synthesis method + metrics of utility and privacy to be calculated).

2. Run `make` in the top level QUIPP-pipeline directory.
    -  This will run a synthesis for each json input file found in `run-inputs/`.  
    -  This also run the generation of a toy dataset and the subsequent synthetis of it. 

    When the pipeline is run, additional directories are created:

     - `generator-outputs`: Sample generated input data (using
   `generators`)

     - `synth-output`: Contains the result of each run (as specified in
   `run-inputs`), which will typically consist of the synthetic data
   itself and a selection of utility and privacy scores

3. `make clean` removes all synthetic output and generated data.
## Methods implemented

### CTGAN


[CTGAN](https://pypi.org/project/ctgan/) is a library that implements the GAN-based Deep Learning data synthesizer for table data which was presented at the NeurIPS 2020 conference by the paper titled [Modeling Tabular data using Conditional GAN](https://arxiv.org/abs/1907.00503).

In order to use CTGAN, the input data must be in either a numpy.ndarray or a pandas.DataFrame object with two types of columns:

- Continuous Columns: can contain any numerical value.
- Discrete Columns: contain a finite number values, whether these are string values or not.


### Privbayes


Given a dataset *D*, PrivBayes first constructs a Bayesian network *N* , which (i) provides a succinct model of the correlations among the attributes in *D* and (ii) allows us to approximate the distribution of data in *D* using a set *P* of low-dimensional marginals of *D*. After that, PrivBayes injects noise into each marginal in *P* to ensure differential privacy and then uses the noisy marginals and the Bayesian network to construct an approximation of the data distribution in *D*. Finally, PrivBayes samples
tuples from the approximate distribution to construct a synthetic dataset (description taken from the paper: [PrivBayes: Private Data Release via Bayesian Networks](https://dl.acm.org/doi/10.1145/3134428)). 

QUiPP uses the PrivBayes implementation within the DataSynthesizer fork found [here](https://github.com/gmingas/DataSynthesizer). in here DataSynthesizer learns a diferentially private Bayesian network capturing the correlation structure between attributes, then draw samples from this model to construct the result dataset. 

DataSynthesizer supports four data type (integer, float, datetime and string). The system allows users to explicitly specify attribute data types. If an attribute data type is not specied by the user, the systems infers it (details about this in  this [document]( https://github.com/gmingas/DataSynthesizer/blob/master/docs/cr-datasynthesizer-privacy.pdf) . DataSynthesizer
allows users to specify a data type, and state whether an
attribute is categorical or continious, but is my understanding that the methods is not directly compatible with continuous features, it converts them to ordinal categorical variables via a histogram.


## Synthpop 


[Synthpop](https://CRAN.R-project.org/package=synthpop) is an R package for producing synthetic
microdata using sequential modelling.

See also the [accompanying paper](https://www.jstatsoft.org/article/view/v074i11).


### SGF

https://vbinds.ch/sites/default/files/PDFs/VLDB17-Bindschaedler-Plausible.pdf

### Baseline methods


## Metrics implemented

### Privacy 

- discloure risk
- leaky output -> calculates disclosure risk 

### Utility


## Reflections on what is next

- Two main streams of work.
- Needs a tutorial (something similar to the census)
- Needs a new harmonisation branch.
- Membership inference attacks

# Random notes for the moment
## Main branches

- census 2011 microdata
    - notebooks with evaluation of the census microdata
- develop-paper 
    - implementation of another utility metric



## Notes

- Lots of placeholders...
- Toy datasets with no clear evaluation
- In clear need of some cleaning. 
- Census dataset seems to be best documented as for a case study.

- Develop paper -> has some more implementations of utilities.
- k = 1 -> priv bayes

## Ideas for a tutorial

Based on these notebooks:
- [Data Generation with Census 2011](https://github.com/alan-turing-institute/QUIPP-pipeline/blob/2011-census-microdata/examples/2011-census-microdata/Census%202011%20Microdata%20-%20Synthetic%20Data%20Generation%20with%20QUIPP.ipynb)
- [Census 2011 Synthetic datasets comparison](https://github.com/alan-turing-institute/QUIPP-pipeline/blob/2011-census-microdata/examples/2011-census-microdata/Synthetic%20datasets%20comparison.ipynb)

### Outline

- How to create the datasets (for all avalaible methods)
- Information of time to run each method.
- Comparison of distributions
- Exposure of metrics.
- 
## Links

- [Synthpop library](https://rdrr.io/cran/synthpop/)
- [Inference from fitted models in synthpop](https://rdrr.io/cran/synthpop/f/inst/doc/inference.pdf)
- [Notes from Callum about QUIPP](https://hackmd.io/tpy3YfaARPWYLA7Rle_5yQ)


## Develop paper

- Main contributions from this branch that can be rescued to a generic develop branch:
    - Feature importance utility metric
        - With example json files of how to run it.
    - Updates in synth data generation methods  
        - Updates to privbayes
        - All the changes to CTGAN
        - New baselines (bootstrap, resampling, etc).
- Other contribution with lower priority but that could be very useful.
    -  New datasets and json files of how to run them
        -  Note: Framingham dataset was not approved on EAG, might still be in the reepo but mus be removed.
    -  Infrastructure to run pipeline in Azure 
     
- Contributions that are very linked to the paper (and probably not generalisable for a new user)
    - Script that generate input files and runs make for experiments
        - Different privacy values
        - Different baselines (independent column resampling sampling, subsampling).
    - Make file is very based on the needs for the paper 
