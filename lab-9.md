---
tags: ggg, ggg2021, ggg201b
---
# Lab 9 outline - assembly collapse; RNAseq - GGG 201(b) lab, March 4th, 2022


([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-9.md))

Contents:

[toc]

## Assembly collapse - some data exploration

### Titus's results

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
