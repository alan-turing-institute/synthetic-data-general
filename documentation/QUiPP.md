# What is QUIPP

> These notes are a mixture of documentation from QUiPP, discussions with members of the team, extracts from papers, reflections from Camila and [notes](https://hackmd.io/tpy3YfaARPWYLA7Rle_5yQ) written by Callum.

## Description

The QUiPP software is a pipeline for generating synthetic tabular data. QUiPP uses a variety of methods as implemented by several libraries and provides measures of privacy and utility on the resulting released datasets. Whilst the synthetic data generation methods come from external libraries, the privacy and utility metrics have been chosen and implemented by the QUiPP team. 

QUiPP is a makefile-based reproducible pipeline. The pipeline is highly configurable with input json files, that tells the pipeline which dataset to use, synthetic methods to run (as well as providing configuration intrinsic to each method), and then subsequently which privacy and utility metrics to calculate.

The input data is present as two files: a csv file which must contain column headings (along with the column data itself), and a json file describing the types of the columns used for synthesis.

The following figure indicates the full pipeline, as run on an input file called example.json. This input file has keywords *dataset* (the base of the filename to use for the original input data) and `synth-method` which refers to one of the synthesis methods. As output, the pipeline produces:

- A number of **output synthetic datasets** csv files (configurable by the input json file) e.g: synthetic_data_1.csv, synthetic_data_2.csv, ...
- **Privacy metrics** in json files as the disclosure risk privacy score (e.g disclosure_risk.json)
- **Utility metrics** in several json files such as correlation like utility metrics and classification tasks (e.g sklearn_classifiers.json)

![](https://i.imgur.com/rfSyvd1.png)

The file-based design of the pipeline makes it very customisable, allowing it to run in any dataset (as long its format and variable types are properly described on the dataset json file) and any configuration of available methods and assessment metrics. 

The data format and types that can be synthesised do not depend on the QUiPP pipeline, but on the synthesis method to use. 

Furthermore, this design makes the QUiPP extensible. For example, to add a new synthetic data method you just need a new folder with the method name, and a file named `run` (which is typically a python wrapper that picks up the values from the input json and passes them to a python script housing the method). 

Finally, QUiPP can also generate toy datasets that can be used in the pipeline.  This is not part of the main functionality of the pipeline, but can be used for experimentation and was useful in the early days of the development of the software.      

## How to use

The installation instructions are found in the [README](https://github.com/alan-turing-institute/QUIPP-pipeline#local-installation) of the repo. 

To run the pipeline, you need to understand the top-level directory structure which mirrors the data pipeline. Not all directories in there are relevant for running the pipeline, therefore I will only describe the ones that are (a more detailed description of the top-level repo can be found in [here](https://github.com/alan-turing-institute/QUIPP-pipeline#top-level-directory-contents)).

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

2. Run `make` in the top-level QUIPP-pipeline directory.
    -  This will run a synthesis for each json input file found in `run-inputs/`.  
    -  This also run the generation of a toy dataset and the subsequent synthesis of it. 

    When the pipeline is run, additional directories are created:

     - `generator-outputs`: Sample generated input data (using
   `generators`)

     - `synth-output`: Contains the result of each run (as specified in
   `run-inputs`), which will typically consist of the synthetic data
   itself and a selection of utility and privacy scores

3. `make clean` removes all synthetic output and generated data.
## Methods implemented

> Disclaimer: Some descriptions below are extracted directly from the papers describing the methods. 

### CTGAN


[CTGAN](https://pypi.org/project/ctgan/) is a library that implements a synthesiser based on generative adversarial networks (GANs) for table data which was presented at the NeurIPS 2020 conference by the paper titled [Modeling Tabular data using Conditional GAN](https://arxiv.org/abs/1907.00503).

In the paper, the authors found that modelling tabular data poses unique challenges for GANs, causing them to fall short
of other baseline methods. These challenges include the need to simultaneously model discrete and continuous columns, the multi-modal non-Gaussian values within each continuous
column, and the severe imbalance of categorical columns.
To address these challenges, the conditional tabular GAN (CTGAN) method is proposed, which introduces several new techniques: augmenting the training procedure with mode-specific normalization, architectural changes, and addressing data imbalance by employing a conditional generator and training-by-sampling (details of what this means can be found in the paper).

To use CTGAN, the input data must be in either a numpy.ndarray or a pandas.DataFrame object with two types of columns: Continuous (can contain any numerical value), and discrete (contain finite number values, whether these are string values or not).

**Summary on QUiPP implementation**: lastest version of the library in `develop-paper`. Seems to work in several tests datasets.

### Privbayes

Given a dataset *D*, PrivBayes first constructs a Bayesian network *N*, which (i) provides a succinct model of the correlations among the attributes in *D* and (ii) allows us to approximate the distribution of data in *D* using a set *P* of low-dimensional marginals of *D*. After that, PrivBayes injects noise into each marginal in *P* to ensure **differential privacy** and then uses the noisy marginals and the Bayesian network to construct an approximation of the data distribution in *D*. Finally, PrivBayes samples
tuples from the approximate distribution to construct a synthetic dataset (description taken from the paper [PrivBayes: Private Data Release via Bayesian Networks](https://dl.acm.org/doi/10.1145/3134428)). 

QUiPP uses the PrivBayes implementation within the DataSynthesizer fork found [here](https://github.com/gmingas/DataSynthesizer), here DataSynthesizer learns a differentially private Bayesian network capturing the correlation structure between attributes, then draw samples from this model to construct the result dataset. 

DataSynthesizer supports four data types (integer, float, datetime and string). The system allows users to explicitly specify attribute data types. If an attribute data type is not specified by the user, the system infers it (details about this in this [document]( https://github.com/gmingas/DataSynthesizer/blob/master/docs/cr-datasynthesizer-privacy.pdf)). DataSynthesizer
allows users to specify a data type, and state whether an attribute is categorical or continuous, but is my understanding that the method is not directly compatible with continuous features, it converts them to ordinal categorical variables via a histogram.

**Summary on QUiPP implementation**: lastest version of the library in `develop-paper`. Seems to work in several tests datasets.


## Synthpop 

[Synthpop](https://CRAN.R-project.org/package=synthpop) is an R package for producing synthetic
microdata using sequential modelling. 

The synthesizing methods can be parametric and non-parametric. The latter is based on
classification and regression trees (CART) that can handle any type of data. The parametric methods. The 
parametric methods can handle numeric, binary, unordered factor and ordered factor data types. The methods
currently implemented and the data types that they accept are listed in [Table 1 of its accompanying paper](https://www.jstatsoft.org/article/view/v074i11).

The synthetic values of the variables are generated sequentially from their conditional distributions given variables already synthesized. QUiPP input json file allows for the specification of the type of synthesis to be used and the order in which the columns are synthesised. The synthesis can also be done only on the marginals 

**Summary on QUiPP implementation**: lastest version of the library in `2011-uk-census-microdata`. The wrapper around the library seems to be customised for two of the existing QUiPP example datasets such as the Polish or UK census. It is not clear to me at the moment of writing if this will run on a new dataset out of the box. 


### SGF

The SGF library uses a probabilistic model that captures the
joint distribution of attributes. The model is learned from
 data samples and a defined directed acyclic graph (DAG) drawn from the original dataset, where the nodes are the random variables, and the edges represent the probabilistic dependency between them.
 
 The main draw of this method is that offers to generate synthetic data in a privacy-preserving manner by using a mechanism that
enforces **plausible deniability**.

Plausible deniability is a criterion that provides a formal privacy guarantee, notably for releasing sensitive datasets: an output record can be released only if a certain amount of input records are indistinguishable, up to a privacy parameter. This notion does not depend on the
background knowledge of an adversary. 
 
The mechanism works in the following way:

Given a generative model M, dataset D, and privacy parameters k and γ, output a synthetic record y or nothing.
1. Randomly sample a seed record d ∈ D.
2. Generate a candidate synthetic record y = M(d).
3. Invoke a plausible deniability privacy test based on the privacy parameters.
4. If the tuple passes the test, then release y.
Otherwise, there is no output.

The larger the privacy parameter k is, the larger the indistinguishability set for the input data record. Also, the closer
to 1 privacy parameter γ is, the stronger the indistinguishability of the input record among other plausible records.

 More details about Plausible Deniability and the SGF method can be found in [this paper](https://vbinds.ch/sites/default/files/PDFs/VLDB17-Bindschaedler-Plausible.pdf).

In the QUiPP, pipeline the input json file allows for the configuration of privacy parameters. The SGF method only supports discrete variables, thus continuous variables mustbe discretized a priori by the user. QUiPP provides extra scripts to create configuration files (beyond the input json file) needed to set up the DAG graph, discretisation and other synthesis parameters. 

**Summary on QUiPP implementation**: lastest version of the library in `develop`. The wrapper around the library seems to be customised for a purposely generated example datasets. This workflow doesn't seem to be well integrated with the general design of the QUiPP pipeline, it was an early-stage development that was not carried over with the pipeline. However, is the only method with a different privacy implementation to differential privacy. I doubt this will run on a new dataset out of the box. 


### Baseline methods

#### Subsample

Returns a random subsample of entries without replacement from the original dataset.  

#### Bootstrap 

Returns a random sample with replacement of entries from the original dataset.  
#### Ensemble

Seems to refer to the "sample" method from Synthpop.

**Summary on QUiPP implementation**:  lastest version of the library in `develop-paper`. Seems to work in several tests datasets.


## Assessment metrics implemented

### Privacy 

#### Privacy parameter as input

Privacy can be given as an input parameter (whether this is differential privacy or plausible deniability) in the PrivBayes and SGF method implementations. 

#### Disclosure risk

The disclosure risk can be defined as the risk that an intruder can access samples from the original dataset to identify information on an individual on the released dataset. 

Calculating disclosure risks can be relevant when producing partial synthetic datasets, where some columns remain unchanged. QUiPP calculates a number of metrics, such as     `EMRi`,`TMRi`,` TMRi`, `TMRa`,`TMRa`,`EMRi_norm`,`EMRi_norm`, `TMRi_norm`.

>DISCLAIMER: I don't understand how these metrics are calculated, there is no clear documentation around it.

**Summary on QUiPP implementation**:  lastest version of the library in `develop`. Seems to be useful only in very specific cases. 

#### PATHE-GAN

>DISCLAIMER: I don't understand how this library is used within QUiPP, there is no clear documentation around it.

### Utility

#### Classifier metrics

Calculates the performance of different machine learning classifiers trained on the original and released datasets for a specified target variable and then tested on the original. These values are then compared to estimate the utility of the released dataset. Results are saved to .json files and an html report is generated.

**Summary on QUiPP implementation**:  lastest version of the library in `develop-paper`, but it doesn't seem to have many changes from `develop`.


#### Correlation metrics

Several correlation-like utility metrics are calculated for all combinations of columns of the original and released dataset. Results are saved into a  .json file. These can be compared to estimate the utility of the released dataset. 

Metrics are the implementation from the [dyton library](http://shakedzy.xyz/dython/) of the following association variables for categorical variables:

- Cramers_v
- theils_u
- correlation_ratio

**Summary on QUiPP implementation**:  lastest version of the library in `develop-paper`, but it doesn't seem to have many changes from `develop`.

#### Feature importance

The feature importance ranking differences between the original and released datasets are calculated, using a random forest classification model, various feature importance measures and various feature rank/score comparison measures.
The results are saved into a .json file. These can be compared to estimate the utility of the released dataset.

Some of the rank comparison metrics implemented are the following:

- Ranked-biased Overlap 
- Correlated rank similarity metrics
- L2 norm 
- KL divergence

**Summary on QUiPP implementation**:  only implemented in `develop-paper`. Seems to work in several tests datasets.

## Streams of work/functionalities 

The main working branch is `develop`. However, this branch doesn't seem to have any new developments (beyond changes in the documentation) in over a year.

Two branches diverged from `develop` and have important new contributions. The description of QUiPP above tries to include all the newest contributions.

One of the branches is the `2011-census-microdata`, where the census data was generated for the ONS project. This branch contains the most documented example of how QUiPP works in form of notebooks. 

The other branch is `develop-paper` and has extra utility metrics (e.g. feature importance) and updated synthesis methods in respect to `develop` and `2011-census-microdata` (plus other developments specifically made to run benchmarking experiments for a paper locally and in Azure), but not very clear documentation/examples. 


## Final thoughts (critics) on QUiPP

There has been an enormous amount of work dedicated to the QUiPP pipeline, and in this section I do not aim to criticise these efforts done by the team but to provide a view that might help us think about how we need to take QUiPP forward. 

- Several methods have been customised for specific datasets (e.g. the Census dataset or generated data). There is no guaranty that they work on a new one out of the box (this seems to be the case for Synthpop and SGF). 
- Synthetic data generation libraries tend to have several different methods implemented. QUiPP in some cases reduces the potential of these libraries, by only accessing one or a few of its methods.  
- The level of customizability of the pipeline makes it also very difficult to understand even in the parts that are documented. Other areas are not documented at all.
- It feels that for the same effort of understanding a new library and writing a wrapper for QUiPP a user would just rather use the library by itself. Unless QUiPP has something unique to offer.
- It is not clear to me what is the unique thing QUiPP has to offer. QUiPP has not an authoritative definition of how to measure utility/privacy (and probably no one does). Perhaps QUiPP can function better as a benchmarking tool as it seemed to be on the `develop-paper` branch. 
- The more libraries in different programing languages get added to the pipeline the more complicated it is to install and run (e.g. the SGF library is an example of this). A good amount of work has to be done in order to keep compatibilities and the interoperability of the pipeline. 



