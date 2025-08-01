
  

# SNP-stitch: lightweight variant calling for ONT data

<img src="./logo.png" width="400">

SNP-stitch (**SNP**  **s**election by **t**ree-based **i**nference **t**olerating **c**omplex **h**eterogeneity) is a lightweight variant caller designed for use with ONT reads. Given a reference sequence, it uses a random forest regressor model to estimate expected base frequencies at a site in absence of true variation and assigns bases which are more than a tolerance value higher as variants.

  

**Note:** SNP-stitch has been primarily designed for, and tested on, data generated by the [DDNS protocol](https://www.protocols.io/view/direct-detection-of-poliovirus-and-nanopore-sequen-rm7vzbyyxvx1/v6) for poliovirus detection. Provided pre-trained models have been trained to this protocol and performance may vary for other uses. It is possible given data where the source sequence is known with confidence to train new models with SNP-stitch and use these as input.

  

## Installation

SNP-stitch is available from PyPI and can be installed with the following command:

``
pip install snp_stitch
``

Alternatively, the source code in this repository can be cloned and used to build/install locally

````
git clone https://github.com/ZoeVance/snp_stitch.git
cd snp_stitch
pip install .
````

## Usage

### Variant calling

Variant calling requires a reference fasta and reads mapped to the same reference in BAM format. Basic calling is performed as follows:

``
snp_stitch call -b PATH_TO_BAM_FILE -r PATH_TO_REFERENCE_FASTA -o PATH_TO_OUTPUT directory
``


An output directory containing a vcf file of all variants will be produced.

Full arguments for variant calling:

````
usage: snp_stitch call [-h] [-b BAM_PATH] [-r REF] [-o OUTPUT_DIR] [-e TOL] [-f MIN_FREQ] [-m MODELS_PATH] [-p CORES]

  
options:

-h, --help              Show this help message and exit
-b --bam                Path to mapped reads in bam format
-r --reference_fasta    Reference fasta to call variants against
-o --out_dir            Path to directory for output. Default: snp_stitch_output_DATE
-e --tolerance          Tolerance for error when calling variants. Default:0.02
-m --models             Path to dir with model and encoder files
-p --cores              Number of cores to use. Default: 1

````

  
### Model training

Model training requires reads that have been mapped to a reference you can be confident they represent - the samples sequenced should not contain any true variation relative to the reference sequence the reads have been mapped against. There is an option to include a file specifying sites to be excluded from training if variant sites are known.

The training data directory should be structured as follows:

````

training_data/
├── dataset1/
│ 	├──reference1.fa
│ 	├──excluded_sites1.bed (optional)
│ 	└──mapped_reads1/
│ 		├── reads1.bam
│ 		├── reads1.bam.bai
│ 		├── reads2.bam
│ 		└── reads2.bam.bai
└── dataset2/
	├──reference2.fa
	├──excluded_sites2.bed (optional)
	└──mapped_reads2/
		├── reads1.bam
		├── reads1.bam.bai
		├── reads2.bam
		└── reads2.bam.bai

````
Each 'dataset' subdirectory should correspond to a single reference sequence in the fasta file e.g. single chromosomes or single viral genomes

Training is run as follows:

``
snp_stitch train -d PATH_TO_TRAINING_DATA_DIRECTORY
``

Full options below, including those that influence the hyperparameters of the model

````
usage: snp_stitch train [-h] [-d DATA_DIR] [-mo USER_MODELS_PATH] [-mn USER_MODEL_NAME] [-p CORES]
                        [-rt ESTIMATOR_RANGE ESTIMATOR_RANGE] [-rd DEPTH_RANGE DEPTH_RANGE]
                        [-ms MIN_SPLIT_RANGE MIN_SPLIT_RANGE] [-ml MIN_LEAF_RANGE MIN_LEAF_RANGE]
                        [-mi MIN_IMP_RANGE MIN_IMP_RANGE] [-cv FOLDS]

options:
  -h, --help            Show this help message and exit
  -d  --data_dir        Path to datasets to be used in training; must contain reference sequence and mapped reads
  -mo --models_output   Path to dir to store model and encoder files. Default: user_models
  -mn --model_name      Name of subdirectory in models output dir storing this model. Default: new_model
  -p  --cores           Number of cores to use. Default: 1
  -rt --range_trees     Range of estimator counts to consider in the random forest during optimisation. Default 50 175
  -rd --tree_depth      Range of depths to consider in the random forest during optimisation. Default 50 175
  -ms --min_split       Range of values for minimum sample split to consider in the random forest during optimisation. Default: 5 10
  -ml --min_leaf        Range of values for minimum leaf samples to consider in the random forest during optimisation. Default: 3 5
  -mi --min_impurity    Range of values for the minimum impurity decrease to consider in the random forest during
                        optimisation - must be in [0,1]. Default: 0 0.5
  -cv --cv_folds        The number of folds to use for k-folds cross validation during optimisation. Default: 5
````