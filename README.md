# White-winged Wood Duck Genome Assembly and Pairwise Sequentially Markovian Coalescent (PSMC) Evalution

For the Practical Computing (BIOL 6220) Final Project, we used 10x Genomics data from a White-winged Wood Duck (WWWD) to generate a whole genome to be used in a Pairwise Sequentially Markovian Coalescent (PSMC) evaluation.

## *PART ONE*

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
 
  --id	* *A unique run ID string* *
  
  --fastqs	Path of the FASTQ folder generated by supernova mkfastq
  
  --sample	(optional) Can be used to select only a single sample of those specified in the sample sheet supplied to mkfastq
  
  --maxreads	(optional) Downsample if more than the specified number of reads is provided (default: 1.2B reads)
  
  --localcores	(optional) limits concurrent sections of Supernova to use the specified number of cores
  
  --localmem	(optional) limits memory use on shared systems where Supernova may attempt to use more resources than a user is allowed

```

#### STEP 3: supernova mkoutput

Command to generate a FASTA file representing your assembly.

```
example: --style=pseudohap
 The pseudohap style generates a single record per scaffold

```
## *PART TWO*

## Burrows-Wheeler Alignment (BWA) Tool

BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome.  

### BWA-MEM

Recommended for high-quality queries as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads.

```
example: 
 bwa mem ref.fa read1.fq read2.fq > aln-pe.sam
 
WWWD example: 
 bwa mem ../WWWD.1.fasta -t 4 4905-CB-0004_S1_L001_R1_001_val_1.fq.gz 4905-CB-0004_S1_L001_R2_001_val_2.fq.gz > WWWD_510.sam

```
## *PART THREE*

## SAMtools (Sequence Alignment/Map)

Samtools is a set of utilities that manipulate alignments in the BAM format.

### SAMtools view

With no options or regions specified, prints all alignments in the specified input alignment file (in SAM, BAM, or CRAM format) to standard output in SAM format (with no header).

Converting SAM directly to a sorted BAM file.

```
example:
 samtools view -bS file.sam | samtools sort - file_sorted
 
WWWD example:
 samtools view -bS -@4 WWWD_510.sam | samtools sort -  -@4 -o WWWD_510_sorted.bam
  -b Output in the BAM format
  -S Ignore for compatibility with previous samtools versions
```

### SAMtools mpileup

Generate VCF, BCF or pileup for one or multiple BAM files.

```
example: 
 samtools mpileup -C50 -uf ref.fa aln.bam | bcftools view -c -  | vcfutils.pl vcf2fq -d 10 -D 100 | gzip > diploid.fq.gz

WWWD example:
 samtools mpileup -C50 -uf WWWD.1.fasta ./4905-CB-0004/WWWD_510_sorted.bam | bcftools call -c - | vcfutils.pl vcf2fq -d10 -D100 | gzip > WWWD510.diploid.fq.gz  
  -d sets and minimum read depth and -D sets the maximum (recommended to set -d to a third of the average depth and -D to twice)
  diploid.fq.gz is the whole-genome diploid consensus sequence of one individual,  generated by samtools mpileup
```

## *PART FOUR*

## PSMC

### Program fq2psmcfa

transforms the consensus sequence into a fasta-like format where the i-th character in the output sequence indicates whether there is at least one heterozygote in the bin [100i, 100i+100)

```
example: 
 fq2psmcfa -q20 diploid.fq.gz > diploid.psmcfa

WWWD example: 
 fq2psmcfa -q20 WWWD510.diploid.fq > WWWD510.diploid.psmcfa
 ```
 
 ### Program psmc
 
 Program `psmc' infers the population size history. In particular, the `-p' option specifies that there are 64 atomic time intervals and 28 (=1+25+1+1) free interval parameters. The first parameter spans the first 4 atomic time intervals, each of the next 25 parameters spans 2 intervals, the 27th spans 4 intervals and the last parameter spans the last 6 time intervals. The `-p' and `-t' options are manually chosen such that after 20 rounds of iterations, at least ~10 recombinations are inferred to occur in the intervals each parameter spans.

## Sources

* [github/psmc](https://github.com/lh3/psmc) - The psmc model
* [SAMTools](https://davetang.org/wiki/tiki-index.php?page=SAMTools) - SAMTools tutorial
* [SAMTools Manual](http://www.htslib.org/doc/samtools-1.1.html) - SAMTools pipline
* [Burrows-Wheeler Alignment Tool](http://bio-bwa.sourceforge.net/bwa.shtml) - bwa Manual Reference Pages
* [10x Genomics](https://support.illumina.com/sequencing/sequencing_software/bcl2fastq-conversion-software.html) - Overview of De Novo Assembly Software
* [Illumina sequencing](https://github.com/lh3/psmc) - bcl2fastq Conversion Software

## Acknowledgments

* Tim Sackton, PhD
* Christopher Balakrishnan, PhD
* Mathew Mckim Louder, PhD
