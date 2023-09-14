# BBBpeptidePredictor
Version: 2023-08-03

Author: Kyungha Kim (bubblebee@unist.ac.kr)

## Introduction
BBBpeptidePredictor is a tool designed to predict specific blood-brain barrier(BBB)-penetrating peptide sequences. These peptides are amplified 12-mer sequences obtained from a combinatorial library of random 12-mer M13 phage display peptides. The peptide sequences are converted into one-hot encoded matrices, which are input for the penetration-predicting machine learning model based on binary classification. This model allows the prediction of BBB-penetration probabilities and the corresponding class of the 12-mer peptide.

## Step 0. Pre-process the sequencing data
> Use conda to install the trimmomatic, bowtie2, samtools.
````
    $ conda install -c bioconda trimmomatic
    $ conda install -c bioconda bowtie2
    $ conda install -c bioconda samtools
````

1. Make an index file to search the variable regions. (fasta file format)
To index the variable region in the phage amplicon library, use the flanking region sequence around the variable region (for flanking sequence confirmation, please refer to the protocol or manual of the specific phage library product used).

* i.e.
```fasta
>PhageM13_flank5
ACCGATACAA TTAAAGGCTC CTTTTGGAGC CTTTTTTTTG GAGATTTTCA ACGTGAAAAA ATTATTATTC GCAATTCCTT TAGTGGTACC TTTCTATTCT CACTCT
>PhageM13_flank3
GGTGGAGG TTCGGCCGAA ACTGTTGAAA GTTGTTTAGC AAAATCCCAT ACAGAAAATC ATTACTAACG TCTGGAAAGA CGACAA
```

2. Rename the fastq files as "a_[12].fastq" and "b_[12].fastq" for duplicates.
* i.e. Sample-a_1.fastq, Sample-a_2.fastq for duplicate 1 and Sample-b_1.fastq, Sample-b_2.fastq for duplicate 2

3. Run the bash files in '/dataseq_prep/' sequentially by 02.merge-sam.sh  Through this process, sequence trimming is followed by alignment and file merging using bowtie2 and samtools. One bam file from duplicates is created. (i.e. Sample.PhageM13.bam)
   
4. Download the Python codes ('/dataseq_prep/phage_bam-to-nVR_freq.py' & '/dataseq_prep/translate-nVR_freq.py')

5. Next, execute the codes using '/dataseq_prep/03.make-nVR_freq.sh' and '/dataseq_prep/04.make-pVR_freq.sh'. Add the flanking region indexing file path to the bash files.
* Output of 03.make-nVR_freq.sh : sample_name.nVR_freq (Count of identified DNA sequence)
* Output of 04.make-pVR_freq.sh : sample_name.pVR_freq (After translating DNA sequences into peptide sequences, counts and frequencies are displayed.)

## Step 1. Make the input dataset composed of filtered peptide sequences
>pVR_freq file format :
>
>#pseq&emsp;Rank&emsp;Count&emsp;pct_total&emsp;pct_over3&emsp;nseq_count&emsp;nseq_rep&emsp;nseq_rep_count

This step removes the outlier sequences and makes the input dataset composed of 12-mer target sequences.
By adding '.pVR_freq' file path to the jupyter notebook code, '/dataseq_prep/05.Preparing_inputdata.ipynb', outlier sequences in all samples are filtered out. Then we could analyze the peptide enrichment during the sequential biopanning and prepare the datasets as the inputs for prediction model training.

## Step 2. Predict the BBB penetration of peptide using the CNN model
1. Tensorflow package and scikit-learn module dependencies
* Tensorflow (ver. 2.9.0)
> Use conda to install the Tensorflow.
````
    $ conda install tensorflow
````

2. Load the '.csv' input file path to the finalized 10-fold cross-validated CNN model in '/ML_pred/06.Applying_to_MLmodel.ipynb'. The peptide sequences can be converted to the one-hot encoded matrix form and the predicted class and score *(probability of BBB penetration)* would be derived.
* Class 1 means penetrating peptides.
* Class 0 means Not-penetrating peptides.

*To see the 10-fold cross-validated model and the manually optimized hyperparameters,* refer to the 'ML_pred/07.10fold_CV.ipynb'.
