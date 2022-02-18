---
tags: ggg, ggg2022, ggg201b
---

# Lab 7 outline - assembly assembly assembly! - GGG 201(b) lab, Feb 18, 2022

[![hackmd-github-sync-badge](https://hackmd.io/dbArj9feSCunshiY6W1sDQ/badge)](https://hackmd.io/dbArj9feSCunshiY6W1sDQ)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-7.md))

[toc]

## Automating an assembly workflow

### Some starting rules for an assembly Snakefile

```
rule annotate:
    input:
        "{sample}-assembly.fa"
    output:
        directory("{sample}_annot")
    shell: """                                                                
       prokka --prefix {output} {input}                                       
    """

rule assemble:
    input:
        r1 = "{sample}_1.fastq.gz",
        r2 = "{sample}_2.fastq.gz"
    output:
        dir = directory("{sample}_assembly"),
        assembly = "{sample}-assembly.fa"
    shell: """                                                                
       megahit -1 {input.r1} -2 {input.r2} -f -m 5e9 -t 4 -o {output.dir}     
       cp {output.dir}/final.contigs.fa {output.assembly}                     
    """

rule quast:
    input:
        "{sample}-assembly.fa"
    output:
        directory("{sample}_quast")
    shell: """                                                                
       quast {input} -o {output}                                              
    """
```

### A snakemake rule for subsampling reads

