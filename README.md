# metis_project_5
# Detecting Tumor Mutational Signatures with a CNN

A tumor mutational signature is a pattern of changes to the DNA that suggests a common underlying mutational process. 

For more information on this topic see: 

- COSMIC: https://cancer.sanger.ac.uk/cosmic/signatures/index.tt 

- PCAWG Consortium Paper, Alexandrov et al.: https://www.nature.com/articles/s41586-020-1943-3

For this project I wanted to see if I could train a neural network to classify tumor types from short aligned DNA sequences. 

### [Presentation Slides and Overview](https://github.com/Beth526/metis_project_5/blob/main/Presentation.pdf)

The training data was generated from the hg38 human reference genome, The Cancer Genome Atlas MAF files and gnomAD VCF files according to the diagram below. Normal variation with allele frequency greater than 0.001 was added into the sequences probabilistically, but to increase the amount incorporated anything with an allele frequency less than 0.5 actually was given a 50% chance of being incorporated. This resulted in about 20-25% of the normal sequences having variation incorporated. Each tumor sequence had one or more mutations added (if there were multiple mutations from the same patient in the given 100bp window).

![Alignment generation](https://github.com/Beth526/metis_project_5/blob/main/generating_input.jpg)

An issue is that changes added to the sequence after a deletion was added were added on top of that deletion. This should not affect a large amount of the sequences, but is an area for improvement. 

### [Code for generating input data](https://github.com/Beth526/metis_project_5/blob/main/reads_generator.ipynb)

### [EDA of generated data (how many patients, number of mutations)](https://github.com/Beth526/metis_project_5/blob/main/preprocessing.ipynb)

### [Training the CNN, Signature Extraction, and Predictions](https://github.com/Beth526/metis_project_5/blob/main/Simple_CNN.ipynb)

Out of the models tested, a CNN with 2 layers performed best for this task, in terms of tumor type recall.

![Predictions on Normal Test Data](https://github.com/Beth526/metis_project_5/blob/main/more_filters_confusion_normal.jpg)

![Predictions on Tumor Test Data](https://github.com/Beth526/metis_project_5/blob/main/more_filters_confusion.jpg)

Input type | Recall | Precision
------------ | ------------- |  ------------- 
normal  | 0.776 | 0.981
bladder | 0.387 | 0.292
breast | 0.006 | 0.097
colorectal | 0.016 | 0.172
glioma/blastoma | 0.030 | 0.091
lung | 0.305 | 0.210
pancreatic | 0.278 | 0.167
renal | 0.243 | 0.205
prostate | 0.277 | 0.137
skin | 0.592 | 0.286
stomach | 0.171 | 0.170
uterine | 0.216 | 0.199
liver | 0.298 | 0.135

It is possible to see where a CNN was 'looking' when it classified sequences by using the dot product of the last convolutional layer and amplification layer following global pooling (I'm refering to this as 'importance'). I used this feature to extact the 'mutational signatures' or patterns the model associated most with each tumor class.

![Example of CNN heatmp](https://github.com/Beth526/metis_project_5/blob/main/heatmap_example.jpg)

### [More CNN heatmaps](https://github.com/Beth526/metis_project_5/blob/main/CNN_heatmaps.pdf)

The subsequences with 'importance' above 0 could be extracted from the top 1000 sequences classified as each tumor type. 

## [Subsequence patterns extracted](https://github.com/Beth526/metis_project_5/blob/main/Subsequences_important_in_classification.ipynb)

Finally, to try the model out on real data, a ctDNA dataset was formated for input into the model. This was anonymized lung cancer patient exome ctDNA sequencing data (NCBI SRA ID: SRR3706303). To format the data, first the raw paired reads were aligned to hg38 with BWA-MEM, then duplicates were marked with MarkDuplicates, and Samtools view was used to downsample (by 10-fold) and remove quality < 20, cigar consuming query > 55 (consistant with max of 45 for indels in model training data), unaligned, non-primary, or PCR-duplicate reads. Then futher processing was needed to get the reference sequence and incoporate the dashes for indels:

### [Formating ctDNA data](https://github.com/Beth526/metis_project_5/blob/main/ctDNA_seq_formatting.ipynb)

This worked OK, but more reads than expected were called as tumor (10% cvs expected 1%) and liver was the most common call, but the softmax probability sum for lung was highest, suggesting the model had highest confidence in these reads.



### TCGA data used:
MAF (Mutect) files from:

BRCA - Breast Invasive Carcinoma

BLCA - Bladder Urothelial Carcinoma

COAD - Colon Adenocarcinoma

PAAD - Pancreatic Adenocarcinoma

PRAD - Prostate Adenocarcinoma

KIRC - Kidney Renal Clear Cell Carcinoma

GBM + LGG combined - Glioblastoma and Low Grade Glioma

LUAD - Lung Adenocarcinoma

SKCM - Skin Cutaneous Melanoma

LIHC - Liver Hepatocellular Carcinoma

UCEC - Uterine Corpus Endometrial Carcinoma

