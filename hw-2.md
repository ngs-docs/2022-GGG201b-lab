---
tags: ggg, ggg2022, ggg201b
---
# GGG 201(b), Lab Homework 2 - 2022

Due by 10pm, Tuesday March 1st

## 1. Connect to GitHub Classroom and clone the hw2 repository

As per [Lab homework #1](https://hackmd.io/kW_TiOLsQxeQzRlj6C33jg?view), connect to [GitHub classroom for HW #2](https://classroom.github.com/a/0f_Y_SSM) and clone this assignment's repository to farm.

## 2. Edit the Snakefile and save to GitHub

Update the Snakefile to calculate at least three different subset assemblies, choosing estimated coverages between 2x and 60x. The default target for the Snakefile should start from a directory containing only the two read files (`SRR2584857_1.fastq.gz` and `SRR2584857_2.fastq.gz`) and the Snakefile, and compute all three assemblies plus their quast statistics and prokka annotations.

Get the Snakefile working, verify that it works in an empty directory (using `--delete-all-output`), and then commit and push as in homework #1.

## 3. Submit assembly results to this Google form

Submit at least three _different_ assembly entries to [this form](https://docs.google.com/forms/d/e/1FAIpQLSeZlOhKgxGCTg7lgM7bePmEboGpM5-5zon2HPKwLQzyP04G5Q/viewform), which will ask for the following information:
1. Your GitHub ID.
2. The filename of one assemby produced by your Snakefile.
3. The number of reads used in the assembly.
4. Your estimate of the coverage (use 4.5 Mb as the genome coverage).
5. The N50 of the assembly.
6. The total bp in contigs > 1kb for the assembly.
7. The total number of contigs > 1kb for the assembly.
8. The total number of protein coding genes (records in the annotation .faa file) for the assembly.

Note that the total number of lines in a single gzipped FASTQ file can be calculated like so:
```
gunzip -c FILENAME | wc -l
```

and the total number of records in a FASTA file (e.g. the .faa file output by prokka) can be calculated by doing
```
grep ^'>' FILENAME | wc -l
```

## 4. Relax in knowledge of a job well done.

Please do inspect the Snakefile on github via the Web interface to make sure it has all your changes!

## Questions? Problems?

If you run into any trouble with submission, that's ok - just let me know.

You can reach me via e-mail at ctbrown@ucdavis.edu, or in UC Davis slack under @ctitusbrown.
