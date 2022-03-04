---
tags: ggg, ggg2021, ggg201b
---
# Lab 9 outline - assembly collapse; RNAseq - GGG 201(b) lab, March 4th, 2022

[![hackmd-github-sync-badge](https://hackmd.io/447eXyGsT_OmP8ySC05tyA/badge)](https://hackmd.io/447eXyGsT_OmP8ySC05tyA)



([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-9.md))

Contents:

[toc]

## Assembly collapse - some data exploration

I will use the data in: https://github.com/ctb/2022-ggg-201b-assembly-collapse

but unfortunately I couldn't get the binder running :slightly_frowning_face: 

### Exploring Titus's results

I cheated and made a Snakefile that did coverage directly :) -
```
rule all:
    input:
        "SRR2584857_quast",
        "SRR2584857_annot",
        expand("SRR2584857.{cover}C_quast", cover=range(1, 41)),
        expand("SRR2584857.{cover}C_annot", cover=range(1, 41))

rule subset_wc:
    input:
        r1 = "{sample}_1.fastq.gz",
        r2 = "{sample}_2.fastq.gz",
    output:
        r1 = "{sample}.{cover}C_1.fastq.gz",
        r2 = "{sample}.{cover}C_2.fastq.gz",
    params:
        n_lines = lambda w: int(float(w.cover) * 4.5e6 / 100 / 2) * 4
    shell: """
        gunzip -c {input.r1} | head -{params.n_lines} | gzip > {output.r1} || 
true
        gunzip -c {input.r2} | head -{params.n_lines} | gzip > {output.r2} || 
true
    """

```

## RNAseq again

Amanda gave a great introduction to the statistics last week, and I wanted to take over where she left off with the binder.

Start by running [the RNASeq binder](https://github.com/nih-cfde/rnaseq-in-the-cloud/):


[![Binder](https://aws-uswest2-binder.pangeo.io/badge_logo.svg)](https://aws-uswest2-binder.pangeo.io/v2/gh/nih-cfde/rnaseq-in-the-cloud/stable?urlpath=rstudio)


Reminder, to run the analysis --

1) start the binder

2) at the command line, run

```
snakemake -j 4 --use-conda
```

3) open `rnaseq-workflow.Rmd`

4) knit to HTML

### Check out the Snakefile

### What's the output of the Snakefile?

### Let's walk through the RMarkdown file

### Statistical considerations and cutoffs

### Appendix: FastQC and MultiQC

(may cover in lab 10)
