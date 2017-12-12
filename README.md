# WWWD-PSMC Project/Fall Practical Computing BIOL 6220
The 10x Genomics Chromium platform allows for whole genome production using linked-reads, potentially providing high resolution in previously difficult to reconstruct regions of the genome. For this project, we use 10x data from a White-winged Wood Duck to generate a whole genome to be used in a Pairwise Sequentially Markovian Coalescent (PSMC) evaluation.
 
Overview of De Novo Assembly Software

Supernova is a software package for de novo assembly from Chromium Linked-Reads that are made from a single whole-genome library from an individual DNA source. A key feature of Supernova is that it creates diploid assemblies, thus separately representing maternal and paternal chromosomes over very long distances. Almost all other methods instead merge homologous chromosomes into single incorrect 'consensus' sequences. Supernova is the only practical method for creating diploid assemblies of large genomes.

## Burrows-Wheeler Alignment (BWA) Tool

BWA is a software package for mapping low-divergent sequences against a large reference genome, such as the human genome.  


### BWA-MEM

Recommended for high-quality queries as it is faster and more accurate. BWA-MEM also has better performance than BWA-backtrack for 70-100bp Illumina reads.

```
example: 
 bwa mem ref.fa read1.fq read2.fq > aln-pe.sam
 
example: 
 bwa mem ../WWWD.1.fasta -t 4 4905-CB-0004_S1_L001_R1_001_val_1.fq.gz 4905-CB-0004_S1_L001_R2_001_val_2.fq.gz > WWWD_510.sam

```

## SAMtools (Sequence Alignment/Map)

Samtools is a set of utilities that manipulate alignments in the BAM format.

### SAMtools view

With no options or regions specified, prints all alignments in the specified input alignment file (in SAM, BAM, or CRAM format) to standard output in SAM format (with no header).

Converting SAM directly to a sorted BAM file.

```
example:
 samtools view -bS file.sam | samtools sort - file_sorted
 
example:
 samtools view -bS -@4 WWWD_510.sam | samtools sort -  -@4 -o WWWD_510_sorted.bam
  -b Output in the BAM format
  -S Ignore for compatibility with previous samtools versions
```

### SAMtools mpileup

Explain what these tests test and why

```
Give an example
```

## Deployment

Add additional notes about how to deploy this on a live system

## Built With

* [Dropwizard](http://www.dropwizard.io/1.0.2/docs/) - The web framework used
* [Maven](https://maven.apache.org/) - Dependency Management
* [ROME](https://rometools.github.io/rome/) - Used to generate RSS Feeds

## Contributing

Please read [CONTRIBUTING.md](https://gist.github.com/PurpleBooth/b24679402957c63ec426) for details on our code of conduct, and the process for submitting pull requests to us.

## Versioning

We use [SemVer](http://semver.org/) for versioning. For the versions available, see the [tags on this repository](https://github.com/your/project/tags). 

## Authors

* **Billie Thompson** - *Initial work* - [PurpleBooth](https://github.com/PurpleBooth)

See also the list of [contributors](https://github.com/your/project/contributors) who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE.md](LICENSE.md) file for details

## Acknowledgments

* Hat tip to anyone who's code was used
* Inspiration
* etc
