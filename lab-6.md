---
tags: ggg, ggg2022, ggg201b
---

# Lab 6 outline - more assembly and annotation - GGG 201(b) lab, Feb 11, 2022

[![hackmd-github-sync-badge](https://hackmd.io/_hIbwz9xT8KrYcH-yXpuUg/badge)](https://hackmd.io/_hIbwz9xT8KrYcH-yXpuUg)


([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-6.md))

Contents:

[toc]

## Breakout sessions

I'm going to send you off into 5 breakout rooms (randomly assigned) for ~20 minutes.

In these breakout rooms, please do the following:

* try to assemble one or more sentences from the shotgun sequencing data present in the two PDFs posted to the lab 6 announcement in canvas. (~10-15 min)
* choose a spokesperson to report answers to the following questions: (5 minutes)
    * what strategy or strategies did you use to assemble the data?
    * if you had an unlimited supply of undergrads to throw at assembly, what would you tell them to do? what would be the most efficient use of your resources?
    * what kinds of things do you think your assembly strategy would have a hard time recovering from data?
    * what kind of additional data would you like in order to build your assembly?
    * what made/makes assembling english text easier than assembling DNA sequence?

## Discussion of assembly strategies

## Notes on computational resources

Variant calling is _disk_ and _CPU_ intensive, and the mapping / pileup phase can be split across multiple computers.

Assembly is _memory_ intensive (and _sometimes_ CPU intensive). It is much harder to split across multiple computers.

## Back to the command line!

Today we're going to explore the results of our assembly from last week using two common "swiss army knife" tools, BLAST and [bedtools](https://github.com/arq5x/bedtools2). BLAST is used for sequence search, and bedtools is used for genome interval arithmetic.

Onwards!

### Log into farm

Log into farm, request a node, and change into week 5's result directory:
```
srun --nodes=1 -p high2 -t 2:00:00 -c 4 --mem 6GB --pty /bin/bash
```
and then
```
cd ~/ggg201-week5/
```

If you didn't finish running prokka last week, you might want to copy lab5 results over from my directory:
::::spoiler
```
mkdir -p ~/ggg201-week5/
cd ~/ggg201-week5/
unzip -o ~ctbrown/transfer/2022-ggg-201-week5.zip
```
::::

### Install some software

::::info
Make sure you've [already installed **the software from lab 5**](https://hackmd.io/hZg5K9aTTmOlzgJV1HUqqg?view#preparing-for-class) into the assembly environment.
::::

Now let's install [bedtools](https://github.com/arq5x/bedtools2) and NCBI BLAST:

```
conda activate assembly
mamba install -y bedtools blast
```
(mamba should actually say they've already been installed, because they were installed by prokka).

We're going to start by doing a gene search in the assembly, which is a pretty common way to explore and validate and use assembled sequences.

### Getting the CRP gene sequence from Salmonella

Let's grab a gene sequence from uniprot - I found [this](https://www.uniprot.org/uniprot/P0A2T6) by googling 'salmonella crp'.

Go to https://www.uniprot.org/uniprot/P0A2T6, then click on Format at the top, then select 'FASTA'. You'll see some protein sequence:

```
>sp|P0A2T6|CRP_SALTY cAMP-activated global transcriptional regulator CRP OS=Salmonella typhimurium (strain LT2 / SGSC1412 / ATCC 700720) OX=99287 GN=crp PE=1 SV=1
MVLGKPQTDPTLEWFLSHCHIHKYPSKSTLIHQGEKAETLYYIVKGSVAVLIKDEEGKEM
ILSYLNQGDFIGELGLFEEGQERSAWVRAKTACEVAEISYKKFRQLIQVNPDILMRLSSQ
MARRLQVTSEKVGNLAFLDVTGRIAQTLLNLAKQPDAMTHPDGMQIKITRQEIGQIVGCS
RETVGRILKMLEDQNLISAHGKTIVVYGTR
```

Let's put that in a file `crp.faa`: copy the sequence from the Web page, then run
```
nano crp.faa
```
to create the file, and then paste the sequence in, and use CTRL-X, y, ENTER to save.

Now:
```
cat crp.faa
```
should show the same sequence as above.

### Searching for the CRP gene in the annotation with grep

```
grep -i crp SRR2584857_annot/SRR2584857_annot.faa
```

### Searching for the CRP gene in the assembly with BLAST

Let's search for the gene in the assembled contigs first.

Format:
```
makeblastdb -dbtype nucl -in SRR2584857-assembly.fa 
```

Now do a search with a protein query against a nucleotide database:
```
tblastn -query crp.faa -db SRR2584857-assembly.fa
```

### Searching for the CRP gene in the annotation with BLAST

Did it get annotated properly?

```
makeblastdb -dbtype prot -in SRR2584857_annot/SRR2584857_annot.faa
```

```
blastp -query crp.faa -db SRR2584857_annot/SRR2584857_annot.faa
```

### playing with e-value cutoffs

...and thinking about sensitivity and specificity...

```
tblastn -query crp.faa -db SRR2584857-assembly.fa -evalue 1e-6
```

### Using bedtools to get genes sequences out

```
grep CRP SRR2584857_annot/SRR2584857_annot.gff 
```

If we put that information into a gene feature format file (GFF):
```
grep CRP SRR2584857_annot/SRR2584857_annot.gff > crp.gff
```
we can then use bedtools to get the gene sequence out from the assembly
```
bedtools getfasta -fi SRR2584857-assembly.fa -bed crp.gff
```

### 

Suppose we have a file that specifies some features of interest - this might be variants, or something else. (Yes, I should have shown you this back in the variants labs :).

Here I've built a file in [BED Format](https://genome.ucsc.edu/FAQ/FAQformat.html#format1) that specifies a location close to the CRP gene:
```
k119_74 15192   15193   position +
```
Let's pretend that we don't know what genes are close to this position, and see if we can get bedtools to tell us.

Per [this issue](https://github.com/arq5x/bedtools2/issues/235#issuecomment-103776618), we need to remove the trailing FASTA sequences in the GFF file:
```
sed '/^##FASTA$/,$d' SRR2584857_annot/SRR2584857_annot.gff > just-features.gff
```
and then sort it:
```
bedtools sort -i just-features.gff  > sorted-features.gff
```

Now we can use the BED file I created above:

```
cp ~ctbrown/transfer/crp-position.bed .
```
to run a query:
```
bedtools closest -a crp-position.bed -b sorted-features.gff  
```
and you should see the same ol' CRP gene. yay!

This is a common kind of workflow to run through - generate various kinds of intermediate files to get ever closer to a question you actually want to answer.

### Conclusion: working with contigs and annotations

Hopefully that gives you some idea of the things you can do with assembled sequences/contigs and their annotations - search them by text string sequence, and manipulate them by annotated features.

Now let's get back to my favorite topic - automation with snakemake!

## Building an assembly Snakefile

OK, now, finally, let's start building a snakefile to automate the assembly and annotation stages.

Run `nano -ET4 Snakefile` and copy/paste in the stuff below:

```
rule assemble:
    shell: """
        megahit -1 SRR2584857_1.fastq.gz -2 SRR2584857_2.fastq.gz -f -m 5e9 -t 4 -o SRR2584857_assembly
    """
    
rule copy_contigs:
    shell: """
        cp SRR2584857_assembly/final.contigs.fa SRR2584857-assembly.fa
    """

rule quast:
    shell: """
        quast SRR2584857-assembly.fa
    """
    
rule annotate:
    shell: """
        prokka --prefix SRR2584857_annot SRR2584857-assembly.fa
    """
```


