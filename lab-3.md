---
tags: ggg, ggg2021, ggg201b
---
# Lab 3 outline - running variant calling on the HPC. - GGG 201(b) lab, Jan 20, 2022

[![hackmd-github-sync-badge](https://hackmd.io/0NT1QTkJQD-PTPHivcL83Q/badge)](https://hackmd.io/0NT1QTkJQD-PTPHivcL83Q)

Contents:

[toc]

## last week reminder

what we did last week -
- defined an end-to-end workflow that led us from a sample to a set of called variants
- we did this by connecting rules to each other (see [Snakefile](https://github.com/ngs-docs/2022-ggg-201b-variant-calling/blob/week3/Snakefile)).

## Getting started with farm

We're going to be switching over to using the on-campus 'farm' HPC today, instead of the binder we used for the last two weeks.

There are several reasons for this.

* farm supports larger analysis efforts (larger files, longer jobs, more memory) than binder;
* so, it's closer to what you'd actually use for real data analysis;
* and it's also _persistent_, so your files will remain there over time!

It's also part of the on campus infrastructure for supporting your research, although there are limitations on who can use farm. Talk to me about that if you end up needing your own HPC account!

### Logging in - background information

The first thing we need to do is log in. This requires using the ssh protocol, which stands for 'secure shell'.

For background, read: 
[connecting to remote computers with ssh](https://ngs-docs.github.io/2021-august-remote-computing/connecting-to-remote-computers-with-ssh.html) and 
[Executing large analyses on HPC clusters](https://ngs-docs.github.io/2021-august-remote-computing/executing-large-analyses-on-hpc-clusters-with-slurm.html). (We'll be going through the second one in GGG 298.)

To log in, you'll need an account name (please use the `datalab-XX` account you should have received in your e-mail) and a password. You'll need to be at the command line prompt that looks like this:
```
(base) datalab-05@farm:~$ 
```
in order to participate in the rest of the lesson today.

### Instructions for Mac and Linux:

Mac and Linux computers already come with the necessary software installed. You'll want to ssh into the account `datalab-XX@farm.cse.ucdavis.edu`.

Here is [some documentation](https://ngs-docs.github.io/2021-august-remote-computing/connecting-to-remote-computers-with-ssh.html#mac-os-x-using-the-terminal-program).

You can also watch [this video](https://video.ucdavis.edu/media/Connecting+to+Remote+Computers+with+SSH/1_806ws8at) - 
the start of the discussion is at 20:20, and 
27:00 is where the actual login is shown.

### Instructions for Windows:

For Windows, you'll need to download Mobaxterm (or use another ssh client) - we have screenshots and instructions for Mobaxterm [here](https://ngs-docs.github.io/2021-august-remote-computing/connecting-to-remote-computers-with-ssh.html#windows-connecting-to-remote-computers-with-mobaxterm).

You can also watch [this video](https://video.ucdavis.edu/media/Connecting+to+Remote+Computers+with+SSH/1_806ws8at) - see 35:50 on video for the actual login.

## Using farm to run the workflow.

We have to go through a couple of different steps in order to use farm today.

### Getting a compute node

First, when we log into farm.cse.ucdavis.edu, we are logging into what's called the "head node". This is a gateway computer that is used for copying files and doing workflow development, but since it is shared by everyone you don't want to run big analyses on it - that will consume resources that others might need.

Instead, you use the slurm queuing system to ask for a compute node - please see [Executing large analyses on HPC clusters](https://ngs-docs.github.io/2021-august-remote-computing/executing-large-analyses-on-hpc-clusters-with-slurm.html) for more information.

So, run:
```
srun -p high2 -t 2:00:00 --mem=5000 --pty bash
```
and your prompt should change to have a different computer name than 'farm' on it.

### Installing snakemake using conda

We're going to start by using conda/mamba to install the `snakemake` workflow software. (See [Installing software on remote computers with conda](https://ngs-docs.github.io/2021-august-remote-computing/installing-software-on-remote-computers-with-conda.html) for more informatio on conda and mamba.)

Run:
```
mamba install -y snakemake-minimal
```
This will go find the snakemake-minimal package on the Interwebs, grab it, download it, and install it in your account.

### Getting the variant calling repository

Next, grab the [`week3` branch](https://github.com/ngs-docs/2022-ggg-201b-variant-calling/blob/week3/) of the variant calling github repository, [github.com/ngs-docs/2022-ggg-201b-variant-calling/](https://github.com/ngs-docs/2022-ggg-201b-variant-calling/).

```
git clone https://github.com/ngs-docs/2022-ggg-201b-variant-calling \
    ggg201-week3 -b week3
```
This puts the files in that repository in the directory `ggg201-week3`.

Here, git and github are ways of tracking files; read [Keeping track of your files with version control](https://ngs-docs.github.io/2021-august-remote-computing/keeping-track-of-your-files-with-version-control.html) for more information. You'll be encountering git again :). I also teach it in GGG298.

### Running the `call_variants` rule

Now, we're finally read to run the workflow!
```
cd ~/ggg201-week3/
snakemake -j 4 --use-conda call_variants
```
This will:

* install the necessary software (using conda)
* run the rule call_variants, along with all of the rules necessary to run it.

Let's take another saunter through [the Snakefile](https://github.com/ngs-docs/2022-ggg-201b-variant-calling/blob/week3/Snakefile) while this is running!

### Editing files on farm

Unfortunately, on farm we do not have easy access to a Web-based editor like we did on the binder, where we could use RStudio. So instead we need to use a command-line editor.

Below, we're going to use the `nano` command-line editor. If you already know how to use a different program, then you can totally use that; just make sure you've set TAB to insert 4 spaces.

(For more information on editing text files on remote computers, see [this lesson](https://ngs-docs.github.io/2021-august-remote-computing/creating-and-modifying-text-files-on-remote-computers.html).)

Let's edit the Snakefile like so:
```
nano -ET4 -m Snakefile
```
and add, at the very top, the three lines:

```
rule all:
    input:
        "SRR2584857_1.ecoli-rel606.vcf"
```
then hit CTRL-X, enter, yes to save.

What does this do?

It makes it so you can just run `snakemake` without specifying a rule name! Try it --

```
rm SRR2584857_1.ecoli-rel606.vcf
snakemake --use-conda -j 4
```
should regenerate the VCF file!

Here, we are seeing that the _first_ rule in a Snakefile is the **default rule**, which will be run if you don't give Snakemake some other rule name.

We are also seeing that when you want a rule to do nothing but produce a specific file, you can just put the filename(s) you want produced as the `input:` for an otherwise empty rule.

(We _could_ move `call_variants` up to the top of this file, too. But there are Reasons we don't want to. Read on!)

## Adding wildcards to snakemake rules

Right now, every time we want to add a new set of reads (the SRR sample) we will need to add a whole new suite of rules. That seems wrong.

We're going to use _snakemake wildcards_ to generalize rules.

Let's try to generalize the `map_reads` rule: everywhere you see `SRR2584857_1`, replace it with `{sample}`. It should look like this:
```
rule map_reads:
    conda: "env-minimap.yml"
    input: ref="ecoli-rel606.fa.gz", reads="{sample}.fastq.gz"
    output: "{sample}.ecoli-rel606.sam"
    shell: """
        minimap2 -ax sr {input.ref} {input.reads} > {output}
    """
```

Now ask snakemake to regenerate the mapping output file:
```
rm SRR2584857_1.ecoli-rel606.sam
snakemake -j 4 --use-conda SRR2584857_1.ecoli-rel606.sam
```
and it should work!

But what happens if you try to run `map_reads` directly? You'll get an error - "Target rules cannot contain wildcards." This is one of the most frustrating and frequenty snakemake errors you'll get! What's going on?

What's happening here is that snakemake can _guess_ that you want to use the `map_reads` rule when you ask for `SRR2584857_1.ecoli-rel606.sam`, and fill in the wildcards from there. But if you just tell it to run `map_reads`, it justifiably throws up its hands and tells you it doesn't know how to fill in `{sample}`.

Now, let's convert more rules!

```
rule call_variants:
    conda: "env-bcftools.yml"
    input:
        ref="ecoli-rel606.fa",
        bamsort="{sample}.ecoli-rel606.bam.sorted"
    output:
        pileup="{sample}.ecoli-rel606.pileup",
        bcf="{sample}.ecoli-rel606.bcf",
        vcf="{sample}.ecoli-rel606.vcf"
    shell: """
        bcftools mpileup -Ou -f {input.ref} {input.bamsort} > {output.pileup}
        bcftools call -mv -Ob {output.pileup} -o {output.bcf}
        bcftools view {output.bcf} > {output.vcf}
    """
```

```
rule gunzip_fa:
    input:                       
        "ecoli-rel606.fa.gz"
    output:
        "ecoli-rel606.fa"
    shell: """  
        gunzip -c {input} > {output}
    """
```