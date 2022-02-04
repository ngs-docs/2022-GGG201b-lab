---
tags: ggg, ggg2022, ggg201b
---

# Lab 5 outline - variant calling odds and ends; transition to assembly (and annotation). - GGG 201(b) lab, Feb 4, 2022

[![hackmd-github-sync-badge](https://hackmd.io/hZg5K9aTTmOlzgJV1HUqqg/badge)](https://hackmd.io/hZg5K9aTTmOlzgJV1HUqqg)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-5.md))

Contents:

[toc]

## preparing for class

Before or at the very beginning of class, please log into farm and run:
```
mamba create -n assembly -y megahit prokka quast
```
- thank you!

## revisiting the homework

Re [homework #1](https://hackmd.io/kW_TiOLsQxeQzRlj6C33jg?view), let's just revisit the steps:

* logged into farm
* `git clone`: made a copy of a set of files stored in your GitHub account (your "repository")
* `srun` - allocated a specific computer from the larger farm cluster 
* ran and edited and tested your workflow
* `git commit` - save changes locally
* `git push` - send saved changes back to your github repo. **<-- this is what hands in your assignment**

I have access your github repository and will use that to look at your handed-in assignment.

## variant calling vs de novo assembly

revisiting variant calling: assumptions and scope:
- which do we want? SNPs, short indels, long indels/structural variation
- accurate short reads work well for SNPs
- inaccurate long reads vs accurate long reads
- assumes that we have a reference that includes the main things
- value: VEP, population structure, GWAS

but variant calling is almost always _reference based_. It's even in the name - "variant"!

Where do our reference genomes come from?

De novo assembly.

(Diagram stuff ensues HERE.)

## running our first assembly

We're going to assemble a paired-end E. coli sample from the Long Term Evolution Experiment - [SRA run SRR2584857](https://www.ebi.ac.uk/ena/browser/view/SRR2584857?show=reads). It's too big for us to download quickly, so I've provided it locally.

### get the data

```
mkdir ~/ggg201-week5
cd ~/ggg201-week5

cp ~ctbrown/data/ggg201b/SRR2584857_*.fastq.gz .
ls -lh
```
and you should see something like
```
>total 360M
>-rw-rw-r-- 1 datalab-01 datalab-01 180M Feb  4 07:00 SRR2584857_1.fastq.gz
>-rw-rw-r-- 1 datalab-01 datalab-01 180M Feb  4 07:00 SRR2584857_2.fastq.gz
```

### allocate a node

```
srun --nodes=1 -p high2 -t 2:00:00 -c 4 --mem 6GB --pty /bin/bash
```

here we're asking for 4 CPUs, 6 GB of RAM, for 2 hours.

### using a conda environment w/software

we're going to use the following software:

* [megahit](https://academic.oup.com/bioinformatics/article/31/10/1674/177884), a de novo genome assembler for microbial samples; alternatives here would be SPAdes.
* [quast](http://bioinf.spbau.ru/quast) for genome assembly evaluation.
* [prokka](https://github.com/tseemann/prokka) for bacterial and archaeal genome annotation.

In brief, megahit takes a pile of reads and gives you a genome; quast gives you statistics about that genome; and prokka annotates the genome with genes.

---

At the beginning of class I asked you to install all of these; you don't need to run this again if you already have:

```
mamba create -n assembly -y megahit prokka quast
```

and now let's use that conda environment:
```
conda activate assembly
```

### running our first assembly

```
megahit -1 SRR2584857_1.fastq.gz -2 SRR2584857_2.fastq.gz -f -m 5e9 -t 4 -o SRR2584857_assembly
```
this will take about 3 minutes, and produce a directory, `SRR2584857_assembly/`.

let's look at the contents of that directory:
```
% ls -l SRR2584857_assembly
total 3340
-rw-rw-r-- 1 datalab-01 datalab-01     230 Feb  4 07:28 checkpoints.txt
-rw-rw-r-- 1 datalab-01 datalab-01       0 Feb  4 07:28 done
-rw-rw-r-- 1 datalab-01 datalab-01 4562183 Feb  4 07:28 final.contigs.fa
drwxrwxr-x 2 datalab-01 datalab-01      70 Feb  4 07:28 intermediate_contigs
-rw-rw-r-- 1 datalab-01 datalab-01  120250 Feb  4 07:28 log
-rw-rw-r-- 1 datalab-01 datalab-01     948 Feb  4 07:25 options.json
```

The key file here is `final.contigs.fa`; the rest are log files or temporary files.

Let's look at final.contigs.fa:
```
head SRR2584857_assembly/final.contigs.fa
```

...well, that's a lot of DNA sequences...

How do we evaluate assemblies?

Let's start by getting some high level statistics. That's what quast is for.

### Run quast

Let's copy the assembled file out of the subdirectory to make it more convenient to work with...
```
cp SRR2584857_assembly/final.contigs.fa SRR2584857-assembly.fa
```

and then run quast:
```
quast SRR2584857-assembly.fa
```

This will produce a directory `quast_results/` with a lot of files in it. Let's look at the basic text report:
```
cat quast_results/latest/report.txt
```

### assembly metrics and stats explained

You should see something like:

```
All statistics are based on contigs of size >= 500 bp, unless otherwise noted (e.g., "# contigs (>= 0 bp)" and "Total length (>= 0 bp)" include all contigs).

Assembly                    SRR2584857_assembly
# contigs (>= 0 bp)         126                
# contigs (>= 1000 bp)      92                 
# contigs (>= 5000 bp)      68                 
# contigs (>= 10000 bp)     65                 
# contigs (>= 25000 bp)     52                 
# contigs (>= 50000 bp)     33                 
Total length (>= 0 bp)      4557069            
Total length (>= 1000 bp)   4542425            
Total length (>= 5000 bp)   4474835            
Total length (>= 10000 bp)  4451946            
Total length (>= 25000 bp)  4249832            
Total length (>= 50000 bp)  3540628            
# contigs                   103                
Largest contig              326752             
Total length                4549884            
GC (%)                      50.72              
N50                         98799              
N75                         52265              
L50                         16                 
L75                         31                 
# N's per 100 kbp           0.00 
```

A few top-level observations:

* assemblies are often fragmented, especially if they use short-read sequencing. This is because of repeats and/or low coverage.
* you may have an estimate of the total length of your genome from other sources (experimental biology, for example :). Typically your assembly will be either a bit SHORTER than that, for haploid samples (because of repeat collapse/loss) or MUCH MUCH LONGER (because of diploidy or polyploidy).

Given that, most of these numbers should be self-explanatory, except for the N50, N75, L50, and L75.

To calculate these, you need to rank-order the contigs by size (largest to smallest, say) and then pick two ranks:
* the rank at which the sum of the length of all contigs below that rank equals 50% of the length of the whole assembly
* the same, but for 75%

Then:

L50: the number of contigs below the 50% rank
N50: the contig length at the 50% rank
L75: the number of contigs below the 75% rank
N75: the contig length at the 75% rank

These exist because assembly is always interested in producing _long contigs_ from _shorter ones_. Again, that's in the name :). So you want measures that reflect how much of the content of your assembly is in long bits.

Let's consider some extremes:

* if your entire genome is assembled into one contig, then your L50 is 1, and your N50 is the genome length.
* if your reads completely failed to assemble, then your L50 is the number reads, and your N50 is the size of an individual read.

### annotating the assembly

Right now the assembly is just a pile of FASTA sequences. That's not so useful.   Let's run the prokka bacterial / archaeal genome annotator:
```
prokka --prefix SRR2584857_annot SRR2584857-assembly.fa
```
This will produce a directory `SRR2584857_annot`; let's take a look:
```
ls -lh SRR2584857_annot
```

you will see:
```
-rw-rw-r-- 1 datalab-01 datalab-01 1.3M Feb  4 07:44 SRR2584857_annot.err
-rw-rw-r-- 1 datalab-01 datalab-01 1.5M Feb  4 07:44 SRR2584857_annot.faa
-rw-rw-r-- 1 datalab-01 datalab-01 4.1M Feb  4 07:44 SRR2584857_annot.ffn
-rw-rw-r-- 1 datalab-01 datalab-01 4.5M Feb  4 07:41 SRR2584857_annot.fna
-rw-rw-r-- 1 datalab-01 datalab-01 4.5M Feb  4 07:44 SRR2584857_annot.fsa
-rw-rw-r-- 1 datalab-01 datalab-01 9.3M Feb  4 07:44 SRR2584857_annot.gbk
-rw-rw-r-- 1 datalab-01 datalab-01 5.5M Feb  4 07:44 SRR2584857_annot.gff
-rw-rw-r-- 1 datalab-01 datalab-01  62K Feb  4 07:44 SRR2584857_annot.log
-rw-rw-r-- 1 datalab-01 datalab-01  15M Feb  4 07:44 SRR2584857_annot.sqn
-rw-rw-r-- 1 datalab-01 datalab-01 900K Feb  4 07:44 SRR2584857_annot.tbl
-rw-rw-r-- 1 datalab-01 datalab-01 289K Feb  4 07:44 SRR2584857_annot.tsv
-rw-rw-r-- 1 datalab-01 datalab-01  113 Feb  4 07:44 SRR2584857_annot.txt
```
and check out the summary information:
```
cat SRR2584857_annot/*.txt
```
You should see something like:
```
>organism: Genus species strain
>contigs: 126
>bases: 4557069
>CDS: 4205
>rRNA: 8
>repeat_region: 2
>tRNA: 76
>tmRNA: 1
```
so that's a summary of what prokka found. Note that repeat regions in particul are going to be underestimated in number.

What are all the other files?

Well, `SRR2584857_annot/SRR2584857_annot.log` contains a copy of the text output from prokka; you can page through it with `more SRR2584857_annot/SRR2584857_annot.log` (use 'q' to quit).

As for the rest, there are a variety of file formats in which annotations are stored. The ones I have found most useful are:
* `SRR2584857_annot/SRR2584857_annot.faa` - contains protein FASTA sequences for the predicted genes
* `SRR2584857_annot/SRR2584857_annot.ffn` - contains DNA sequences for the predicted genes

although there's a bunch of other potentially useful output in there if you plan on submitting a bacterial genome to Genbank :).

You can read more about prokka [here](https://github.com/tseemann/prokka).

## next class

we'll build a bit of a snakefile.

we'll talk about parameter exploration.

and we'll discuss how assembly works underneath.
