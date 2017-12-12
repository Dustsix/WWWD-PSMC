# White-winged Wood Duck Genome Assembly and Pairwise Sequentially Markovian Coalescent (PSMC) Evaluation

For the Practical Computing (BIOL 6220) Final Project, we used 10x Genomics data from a White-winged Wood Duck (WWWD) to generate a whole genome to be used in a Pairwise Sequentially Markovian Coalescent (PSMC) evaluation. The Pairwise Sequentially Markovian Coalescent (PSMC) model uses information in a  diploid sequence of one individual to infer the historic population changes.

## *PART ONE: Producing a Genome*

## 10x Genomics 

For this project, sample prep, library prep, instrument (Chromium), and sequencing (Illumina HiSeq X) was performed by 10x Genomics. The 10x Genomics Chromium platform produces a whole genome using linked-reads, potentially providing high resolution in previously difficult to reconstruct regions of the genome. This platform has the potential to provide valuable insight for conservation genetics that require comparison between areas with high polymorphisms.
 

### De Novo Assembly Software
Data produced from sequencing is assembled with Supernova, a software package for de novo assembly from Chromium Linked-Reads that are made from a single whole-genome library from an individual DNA source (in this case WWWD number 510). An important feature of Supernova is that it creates diploid assemblies, which represent maternal and paternal chromosomes, any differences in sequence are due to variants between the the maternal and paternal alleles. Other techniques can merge homologous chromosomes into single incorrect consensus sequence. The Supernova software package includes two processing pipelines (STEP 1/STEP 2) and one post-processing pipline (STEP 3).

#### STEP 1: supernova mkfastq 

Wraps Illumina's BCL file to correctly demultiplex Chromium sequencing samples and to convert data to FASTQ (FASTQ format is a text-based format for storing both a nucleotide sequence and its corresponding quality score) for downstream analysis.

#### STEP 2: supernova run 

Takes FASTQ files containing barcoded reads from supernova mkfastq (STEP 1) and builds a graph-based assembly.

```
example: 
 supernova run --id=${SAMPLE}_assembly --fastqs=/$SPECIES/${SAMPLE}_fastq --maxreads=600000000 --localcores 36 --localmem 600
 
  --id      A unique run ID string
  
  --fastqs	Path of the FASTQ folder generated by supernova mkfastq
  
  --sample	(optional) Can be used to select only a single sample of those specified in the sample sheet supplied to mkfastq
  
  --maxreads	(optional) Downsample if more than the specified number of reads is provided (default: 1.2B reads)
  
  --localcores	(optional) limits concurrent sections of Supernova to use the specified number of cores
  
  --localmem	(optional) limits memory use on shared systems where Supernova may attempt to use more resources than a user is allowed

```

#### STEP 3: supernova mkoutput

Generate a FASTA file representing your assembly. There are four styles of FASTA output: raw, megabubbles, pseudohap, pseudohap2. For this data, pseudohap style was used.

```
example: --style=pseudohap
 The pseudohap style generates a single record per scaffold
```

## *PART TWO: Mapping Reads to Assembled Genome*

For PSMC analysis, we needed to map reads to a reference genome. We assume our sequenced reads were low divergence between the subject (the individual reads from WWWW #510) and the reference (the WWWD genome assembly to which the reads will be mapped). 

We are looking for genomic varition in WWWD. Small genomic variants such as SNPs (single nucleotide polymorphisms). By mapping the reads generated in STEP 2 of PART ONE to the assembled diploid genome, we can identify SNPs.


## Burrows-Wheeler Alignment (BWA) Tool

BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the WWWD genome produced in PART ONE. It consists of three algorithms: BWA-backtrack, BWA-SW and BWA-MEM.

### STEP 1: BWA-MEM

Recommended for high-quality as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads (WWWD #510: mean read length = 139.50bp).

```
example: 
 bwa mem ref.fa read1.fq read2.fq > aln-pe.sam
 
WWWD example: 
 bwa mem ../WWWD.1.fasta -t 4 4905-CB-0004_S1_L001_R1_001_val_1.fq.gz 4905-CB-0004_S1_L001_R2_001_val_2.fq.gz > WWWD_510.sam
```
## *PART THREE: Pairwise Sequentially Markovian Coalescent (PSMC) Evaluation*

### SAMtools (Sequence Alignment/Map)

Samtools is a set of utilities that manipulate alignments in the SAM format. 
 - Sequence Alignment/Map (SAM) format for alignment of nucleotide sequences to reference sequence. 
 - Binary Alignment Map (BAM) is the comprehensive raw data of genome sequencing.

### Step 1: SAMtools view

Converting SAM directly to a sorted BAM file. 

```
example:
 samtools view -bS file.sam | samtools sort - file_sorted
 
WWWD example:
 samtools view -bS -@4 WWWD_510.sam | samtools sort -  -@4 -o WWWD_510_sorted.bam
  -b Output in the BAM format
  -S Ignore for compatibility with previous samtools versions
```

### Step 2: SAMtools mpileup

This uses the sorted bam file and WWWD reference genome, generates an mpileup using samtools, calls the consensus sequence with bcftools, and then filters and converts the consensus to fastq format, writing the results for each chromosome to a separate fastq file. 

```
example: 
 samtools mpileup -C50 -uf ref.fa aln.bam | bcftools view -c -  | vcfutils.pl vcf2fq -d 10 -D 100 | gzip > diploid.fq.gz
 
 samtools:
  --Q and -q in mpileup determine the cutoffs for baseQ and mapQ
  --v tells mpileup to produce vcf output, and -u says that should be uncompressed
  --f is the reference fasta used (needs to be indexed)
  --r is the region to call the mpileup

bcftools:
  --c calls a consensus sequence from the mpileup using the original calling method 

vcfutils.pl:
  --d and -d determine the minimum and maximum coverage to allow for vcf2fq, anything outside that range is filtered
  --Q sets the root mean squared mapping quality minimum

WWWD example:
 samtools mpileup -C50 -uf WWWD.1.fasta ./4905-CB-0004/WWWD_510_sorted.bam | bcftools call -c - | vcfutils.pl vcf2fq -d10 -D100 | gzip > WWWD510.diploid.fq.gz  
  
  --d sets and minimum read depth and -D sets the maximum (recommended to set -d to a third of the average depth and -D to twice)
```

### Step 3: fq2psmcfa

Transforms the consensus sequence into a fasta-like format where the i-th character in the output sequence indicates whether there is at least one heterozygote.

```
example: 
 fq2psmcfa -q20 diploid.fq.gz > diploid.psmcfa

WWWD example: 
 fq2psmcfa -q20 WWWD510.diploid.fq > WWWD510.diploid.psmcfa
 ```
 
 ### Step 4: psmc
 
 Program psmc infers the population history. 
  -p option specifies that there are 64 atomic time intervals and 28 (=1+25+1+1) free interval parameters. 
  -p and -t  are manually chosen such that after 20 rounds of iterations, at least ~10 recombinations are inferred to occur in the intervals each parameter spans.
 
 ```
example: 
 psmc -N25 -t15 -r5 -p "4+25*2+4+6" -o diploid.psmc diploid.psmcfa

WWWD example: 
 psmc -N25 -t15 -r5 -p "4+25*2+4+6" -o WWWD510.diploid.psmc WWWD510.diploid.psmcfa
 ```
There are various ways of rescaling the time and the popuation size more meaningful PSMC (output is scaled to the 2N_0).

## Sources

* [github/psmc](https://github.com/lh3/psmc) - The psmc model
* [SAMTools](https://davetang.org/wiki/tiki-index.php?page=SAMTools) - SAMTools tutorial
* [SAMTools Manual](http://www.htslib.org/doc/samtools-1.1.html) - SAMTools pipline
* [Burrows-Wheeler Alignment Tool](http://bio-bwa.sourceforge.net/bwa.shtml) - bwa Manual Reference Pages
* [10x Genomics](https://support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) - Overview of De Novo Assembly Software
* [Illumina sequencing](https://github.com/lh3/psmc) - bcl2fastq Conversion Software
* [PSMC](https://informatics.fas.harvard.edu/psmc-journal-club-walkthrough.html) - PSMC journal club walkthrough

## Acknowledgments

* Tim Sackton, PhD
* Christopher Balakrishnan, PhD
* Mathew Mckim Louder, PhD
