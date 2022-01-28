---
tags: ggg, ggg2021, ggg201b
---
# Lab 4 outline - even more variant calling; snakemake wildcards. - GGG 201(b) lab, Jan 29, 2021

[![hackmd-github-sync-badge](https://hackmd.io/Wldqc4MiTWWeh-W6OIHMQA/badge)](https://hackmd.io/Wldqc4MiTWWeh-W6OIHMQA)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-4.md))

Contents:

[toc]

## last week outline + additions

last Friday was mostly "what are we doing with variant calling, and this workflow specifically?" And we started to talk about doing a better job of variant calling, and/or filtering our variant calls.

We'll continue (finish?) that discussion today.

## but first... homework.

I posted some homework! See [lab HW #1](https://hackmd.io/kW_TiOLsQxeQzRlj6C33jg?view.)

It is due next Thursday at 10pm. Please let me know if that is a terrible due date. (We're going to move on to assembly next Friday so I would like to have get the variant calling homework out of the way for lab 5!)

A lot of this homework is about getting things basically working - github, editing, making changes to Snakefiles.

key note: you can work together but please do hand things in separately.

## Log into farm, etc.

Log into farm with your datalab-XX account, as we did [last week](https://hackmd.io/0NT1QTkJQD-PTPHivcL83Q?view#Getting-started-with-farm).

Start up a 2 hour session on a compute node:
```
srun -p high2 -t 2:00:00 -c 2 --mem=5000 --pty bash
```

and get the week4_begin branch of the variant calling repository.

```
git clone https://github.com/ngs-docs/2022-ggg-201b-variant-calling \
    ggg201-week4 -b week4_begin
```

and change in to that directory
```
cd ~/ggg201-week4/
```
and then run things:
```
snakemake -p -j 2 --use-conda
```

Note that you don't need to install snakemake again, since you already did that once (last week). Revisit the `mamba install` instructions from lab 3 if you haven't run them yet.

## revisiting the end of last week (week 3)

We're starting from [this Snakefile](https://github.com/ngs-docs/2022-ggg-201b-variant-calling/blob/week4_begin/Snakefile), which has split out the gunzip step, but has not split out the VCF file.

(This is the same Snakefile you will be starting with in the homework.)

Let's start by adding in a new VCF output file - `.spec.vcf`, for "specific."

1. in rule `all`, rename the current input `SRR*.ecoli-rel606.vcf` to have `.sens.vcf` on the end.
2. Add a new input with `.spec.vcf` on the end to rule `all`.
3. in rule `call_variants`, make the same change to the output VCF.
4. add a new rule, `call_variants_spec`, that we will use to implement filtering.

```
rule call_variants_spec:
    conda: "env-bcftools.yml"
    input:
        vcf="SRR2584857_1.ecoli-rel606.sens.vcf"
    output:
        vcf="SRR2584857_1.ecoli-rel606.spec.vcf"
    shell: """
        bcftools filter -Ov {input} > {output}
    """
```

Now re-run everything --
```
snakemake -j 1 --use-conda
```
and make sure you have two output VCF files:
```
ls *.spec.vcf *.sens.vcf
```
should show
```
>SRR2584857_1.ecoli-rel606.sens.vcf  SRR2584857_1.ecoli-rel606.spec.vcf
```

### Actually doing some filtering

These two VCF files are identical - nothing we're doing above does any filtering. Let's change that.

Let's look at the end of one of the files -

```
tail -1 *.sens.vcf
```

Each line has this format:
```
ecoli   4561001 .       G       T       9.88514 .       \
DP=1;SGB=-0.379885;MQ0F=0;AC=2;AN=2;DP4=0,0,0,1;MQ=60    \
GT:PL   1/1:39,3,0
```

The 6th column is the QUALity, the 6th is the INFO field, and the 8th is genotyping information.

The INFO field in particular contains all sorts of juicy info - let's take a look [at the docs for some of the common fields](https://en.wikipedia.org/wiki/Variant_Call_Format#Common_INFO_fields). We'll be using DP, depth, below.

We can filter on these fields by specifing a conditions string to `bcftools filter` and then saying we either want to include (`-i`) or exclude (`-e`) variants that match those conditions. Stealing from [Torsten Seeman's blog post](http://thegenomefactory.blogspot.com/2018/10/a-unix-one-liner-to-call-bacterial.html), let's exclude calls that:

* have low quality (qual < 40)
* have low depth (dp < 10)
* "look" heterozygous (GT != 1/1), where GT stands for genotype (see 8th column).

To do this, let's add `-e 'QUAL<40 || DP<10 || GT!="1/1"'` to the `bcftools filter` call for the 'specific' one.

Your rule should look like this:
```
rule call_variants_spec:
    conda: "env-bcftools.yml"
    input:
        vcf="SRR2584857_1.ecoli-rel606.sens.vcf"
    output:
        vcf="SRR2584857_1.ecoli-rel606.spec.vcf"
    shell: """
        bcftools filter -Ov -e 'QUAL<40 || DP<10 || GT!="1/1"' {input} > {output\
}   
    """
```

Now run
```
snakemake -j 1 --use-conda -p
```
and see what happens.

::::spoiler
Oops! We forgot to ask snakemake to regenerate the file! Snakemake doesn't know that you've changed the parameters, so we need force it to regenerate the spec vcf.
```
rm -f *.spec.vcf
snakemake -j 1 --use-conda -p
```
::::

### Digression: why do things this way?

Why are we calling variants sensitively, then filtering afterwards?

More specifically,
* why are we calling variants with 'sens' where we know that the false positive rate will be high?
* why aren't we just calling variants twice?

::::warning
Because...
::::spoiler
1. You often want to "bookend" your parameters this way - e.g. here we are doing very sensitive variant calling, and very specific variant calling. Presumably the sensitive one will have very false negatives (will miss very little, but may be overwhelming), and the specific one will have very few false positives (but may miss a lot).
2. The pileup step is expensive. It's best to run it just once.
::::

## Adding more depth

But now, we have a problem. What is it?

It turns out that we didn't call any variants!
```
tail -1 *.spec.vcf
```
shows just comment lines. What happened?

Well, I gave you a cut-down data file... which is good practice when you're developing a pipeline!

Look at the current file size -
```
ls -lh SRR2584857_1.fastq.gz
```
and remember it - then, let's get a bigger data set!

The full data sets 
are under
https://osf.io/vzfc6/. Go find the folder  `2020... variant calling... big`, and then select the relevant data set. Now find the new URL, and replace it in the Snakefile.

Your new download rule should look like this:

```
rule download_data:
    conda: "env-wget.yml"
    output: "{sample}.fastq.gz"
    shell: """                                                                   
        wget https://osf.io/aksmc/download -O {output}                           
    """
```

Make those changes and now run
```
snakemake -j 1 -p --delete-all-output
```
(why do we need to do this?)

and then
```
snakemake -j 1 -p --use-conda
```

Does that work better? How do we tell?

<!-- doing a tview
install samtools?
-->


