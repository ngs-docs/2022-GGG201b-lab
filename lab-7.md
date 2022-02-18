---
tags: ggg, ggg2022, ggg201b
---

# Lab 7 outline - assembly assembly assembly! - GGG 201(b) lab, Feb 18, 2022

[![hackmd-github-sync-badge](https://hackmd.io/dbArj9feSCunshiY6W1sDQ/badge)](https://hackmd.io/dbArj9feSCunshiY6W1sDQ)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-7.md))

[toc]

## Homework #1 review

Everyone did great!

The most common problem was not putting any filtration rules in the `bcftools filter` command, so the sens and spec VCF files were identical. Detecting that involved comparing or inspecting the VCF output; I did a quick check like so,
```
grep -v ^# VCF_FILE | wc -l
```

Other than that, most other problems involved confused Snakefiles and could have been detected by:
```
snakemake -j 2 --delete-all-outputs
snakemake -j 2
```

The trickiest problem I saw was where people were not downloading the right FASTQ data files. It turns out that if you don't have a good FASTQ file, everything still runs without error - you just don't get any variants!! So you've gotta look at the variant call file.

tl;dr always look at the file sizes of the inputs, and always inspect the outputs!

## 'expand' vs wildcards

Consider:

(a)
```
rule all:
    input:
        expand("{sample}.ecoli-rel606.vcf",
            sample=['sample1', 'sample2'])
```

(b)
```
rule all:
    input:
        "sample1.ecoli-rel606.vcf",
        "sample2.ecoli-rel606.vcf"
```

(c)
```
rule all:
    input:
        "{sample}.ecoli-rel606.vcf"
```

which of these three will work?

::::spoiler
(a) and (b) are identical - the 'expand' produces a set of concrete targets.

(c) is fishy because wildcards must always appear in input and output both.

`expand` usually only shows up in rules with only an `input:` block.
::::

## More assembly!

### Log into farm

Log into farm, request a node, and change into week 5's result directory:
```
srun --nodes=1 -p high2 -t 1:00:00 -c 4 --mem 6GB --pty /bin/bash
```
activate the `assembly` conda environment:
```
conda activate assembly
```
and then change to the right directory:
```
cd ~/ggg201-week5/
```

If you didn't finish running prokka two weeks ago, you might want to copy lab5 results over from my directory:
::::spoiler
```
mkdir -p ~/ggg201-week5/
cd ~/ggg201-week5/
unzip -o ~ctbrown/transfer/2022-ggg-201-week5.zip
```
::::

## Automating an assembly workflow

You'll probably need to install `snakemake-minimal`, first:
```
mamba install snakemake-minimal
```

while that's going on, let's look at...

### Some starting rules for an assembly Snakefile

(Based on [lab 5](https://hackmd.io/hZg5K9aTTmOlzgJV1HUqqg?view).)

```
rule annotate:
    input:
        "SRR2584857-assembly.fa"
    output:
        directory("SRR2584857_annot")
    shell: """                                                                
       prokka --prefix {output} {input}                                       
    """

rule assemble:
    input:
        r1 = "SRR2584857_1.fastq.gz",
        r2 = "SRR2584857_2.fastq.gz"
    output:
        dir = directory("SRR2584857_assembly"),
        assembly = "SRR2584857-assembly.fa"
    shell: """                                                                
       megahit -1 {input.r1} -2 {input.r2} -f -m 5e9 -t 4 -o {output.dir}     
       cp {output.dir}/final.contigs.fa {output.assembly}                     
    """

rule quast:
    input:
        "SRR2584857-assembly.fa"
    output:
        directory("SRR2584857_quast")
    shell: """                                                                
       quast {input} -o {output}                                              
    """
```

Put these in a Snakefile and try running each of these by name to see if there are any syntax errors:
```
snakemake -j 1 assemble
snakemake -j 1 annotate
snakemake -j 1 quast
```

::::info
How do we count the number of genes in an assembled genome?
::::

TODO:
* how would we add in wildcards here?
* what's a good default rule, i.e. what targets should go in it?


::::info
Your prokka annotation may not run depending on what version you have installed. If you see that 'hmmer' fails, you may need to run the following commands:
```

mamba install -y perl-app-cpanminus
cpanm Bio::SearchIO::hmmer --force
```
see [this github issue](https://github.com/tseemann/prokka/issues/528#issuecomment-1035198733) for more information.
::::

### A snakemake rule for subsampling reads

```
rule subset_100k:
    input:
        r1 = "SRR2584857_1.fastq.gz",
        r2 = "SRR2584857_2.fastq.gz",
    output:
        r1 = "SRR2584857.sub100k_1.fastq.gz",
        r2 = "SRR2584857.sub100k_2.fastq.gz",
    shell: """                                                                
        gunzip -c {input.r1} | head -400000 | gzip > {output.r1} || true      
        gunzip -c {input.r2} | head -400000 | gzip > {output.r2} || true      
    """
```

what's going on here?

::::info
How do we count the number of reads in a file?

And how do we turn that into an estimate of coverage?
::::
