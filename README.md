## GATK-Joint-Genotyping-Pipeline

### More efficient way to run GATK4's HaplotypeCaller | GenotypeGVCFs pipeline for RNAseq SNP data

This was configured for my personal use. You will need to change the path names, sample names, etc.
Due to the slow nature of GATK's CombineGVCFs | GenotypeGVCFs pipeline,
this script uses a tactic to reduce the dataset to just the SNPs of interest,
(identified by first running HaplotypeCaller on pooled samples),
and then running the joint genotyping pipeline on individual samples at just those SNPs.
It uses the GenomicsDBImport | GenotypeGVCFs, which as of today (May 2, 2018),
only works on one interval (chromosome/contig) at a time.
This script will make an artificial interval for GenomicsDBImport | GenotypeGVCFs,
and then restore the original CHROM/POS identities afterwards.  
**NOTE:** Since I only had 3 samples ("229", "230", "231"), I did not parallelize by sample. If you have many samples you may want to do that. As written however, you'll want to run Step 1, then all samples in Step 2 simultaneously, Step 3, Step 4 (if desired).

### This pipeline is designed to:
1. Call SNPs in a pooled transcriptome
2. Genotype (efficiently) the same SNPs in each sample (3 in this case: 229, 230, 231)
3. Annotate the SNPs for effects such as synonymous/missense, etc.
4. Merge genotypes, SNP annotations, and gene annotations from a trinotate report

Prepare pooled (3) samples for HaplotypeCaller:  
`sbatch variantcallingALL.sh`

Call HaplotypeCaller on pooled (3) samples...  
Also make sure the output includes only biallelic SNPs:  
`sbatch variantcallingALL_part2.sh`

Prepare Sample 229 for HaplotypeCaller:  
`sbatch variantcalling229.sh`

Call HaplotypeCaller on Sample 229:  
`sbatch variantcalling229_part2.sh`

Prepare Sample 230 for HaplotypeCaller:  
`sbatch variantcalling230.sh`

Call HaplotypeCaller on Sample 230:  
`sbatch variantcalling230_part2.sh`

Prepare Sample 231 for HaplotypeCaller:  
`sbatch variantcalling231.sh`

Call HaplotypeCaller on Sample 231:  
`sbatch variantcalling231_part2.sh`

Create fasta of concatenated REF snps from pooled output (use VCF file)  
Call this `concat.fa`  
Create table of desired SNPs from pooled output (use VCF file)  
Columns should be `CHROM, POS, NEW, REF`
Call this `SNPlist_concat.txt`  

Change names of samples in Samples 229, 230, 231 VCF files  
`sbatch changeVCFnames.sh`

Run joint genotyping pipeline create here...  
This sidesteps the ENORMOUSLY inefficient pipeline CombineGVCFs | GenotypeGVCFs  
With 100,000s contigs, estimated waiting times are 10-60 years!  
The pipeline in GATK 4 of GenomicsDBImport | GenotypeGVCFs is much faster...  
Unfortunately right now it only accepts one interval (contig)  
This pipeline below will use the SNPs identified in the pooled VCF...  
And create a single (fake) interval from them, then run GenomicsDBImport | GenotypeGVCFs  
Afterwards it will return the intervals to their original identities  
It will make sure the output only includes biallelic SNPs  
`sbatch genotypecalling_fast.sh`

Now do some final editing: make sure REF/ALT for SNPs agree between pooled and individual genotypes:  
e.g., Run this in Rstudio  
`final.editing.R`

Next run TransDecoder to get GFF3 of CDS:  
`sbatch Transdecoder_supertranscripts.sh`

Finally, run VEP. I suggest using table output  
`sbatch vep_tableoutput.sh`

If you have a trinotate report in *.txt format (use NA instead of . for missing values)...  
You can run this script in Rstudio to combine genotypes, SNP annotations, and gene annotations  
You might need to edit a few things to match your files:  
`JoinGenotypesAndSnpAnnotations.R`
