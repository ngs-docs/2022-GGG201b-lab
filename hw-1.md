---
tags: ggg, ggg2022, ggg298
---
# GGG 201(b), Lab Homework 1 - 2022

[![hackmd-github-sync-badge](https://hackmd.io/kW_TiOLsQxeQzRlj6C33jg/badge)](https://hackmd.io/kW_TiOLsQxeQzRlj6C33jg)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/hw-1.md))

Due by 10pm, Thursday Feb 3rd

Please note that you may freely work together with others, but you must hand it in separately.

## 1. Sign up for GitHub Classroom

(Estimate: 10 min)

Sign up for GitHub Classroom using [this link](https://classroom.github.com/a/0UWAZXX_). This may involve creating a (free) GitHub account.

## 2. Log into farm & clone your github repo for hw1

(Estimate: 5 minutes)

On farm, clone your assignment repository, which will be something like `https://github.com/ngs-docs/2022-ggg-201b-lab-hw1-ctb`; do so like this,

```
git clone YOUR_REPO_URL 201b-lab-hw1
```

Change into that directory:
```
cd ~/201b-lab-hw1/
```

and start up a 2 hour session on a compute node:
```
srun -p high2 -t 2:00:00 -c 4 --mem=10000 --pty bash
```

**You'll need to change directories and use srun every time you log in to farm.**

## 3. Run the current workflow

(Estimate: 15 minutes)

Fix the Snakefile by updating from the very latest version:
```
curl -L -O https://raw.githubusercontent.com/ngs-docs/2022-ggg-201b-hw1/main/Snakefile
```
Note, you'll only need to do this once.

Then, make sure you can successfully run the current workflow:

```
snakemake -j 1 --use-conda -p
```
which should produce a file `SRR2584857_1.ecoli-rel606.vcf`.

## 4. Edit and update your Snakefile

::::info
You'll need to edit your Snakefile in a text editor. We used nano in class; see [lab 3 notes](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/lab-3.md). You can read more about editing files on a remote computer [here](https://ngs-docs.github.io/2021-august-remote-computing/creating-and-modifying-text-files-on-remote-computers.html), and use options other than nano.
::::

---

1. Reprise the generation of two VCF files from end of lab 3 (and lab 4) in your Snakefile. One output VCF file should be named with the suffix `.sens.vcf` (for "sensitive"), and the other with the suffix `.spec.vcf` (for "specific"). The specific VCF file should be created from the sensitive variant file with the VCF filtering parameters in [this blog post](http://thegenomefactory.blogspot.com/2018/10/a-unix-one-liner-to-call-bacterial.html).

2. Add three more samples to the Snakefile, and adjust the current sample to use the full set of reads.

For (2), please visit [the data repository for the course](https://osf.io/vzfc6/) and locate the four data sets in the folder under `2020... variant calling... big`. All four sample sets start with `SRR2584...`. The download URLs for them can be determined by clicking on each data set and copying the URL.

Add them to your Snakefile so that (a) running `snakemake -j 1 --use-conda` without any targets automatically downloads them and (at the end of the workflow) generates two sets of VCF files for each data set.

The simplest way to do this is to create four different download_data rules (with four different names), each one adjusted to download one of the four files from the different URL and save it as the correct filename.

::::warning
Note: if you are feeling adventurous you can
use trick #1 from [this blog post](http://ivory.idyll.org/blog/2020-snakemake-hacks-collections-files.html) to do this in one rule, instead of using four.
::::

Please make sure of the following:

* Running `snakemake -j 1 --use-conda -p` by itself should go from raw data to final results for all samples;
* All of the generated files (including intermediates) are in at least one 'output:' annotation, so that e.g. `snakemake --delete-all-output` removes all of the generated files in the directory.

## 5. Commit and push your changes back to github.

At any time, do the following to save changes.

```
git commit -am "updated Snakefile"
git push origin main
```
(you can do this as many times as you want, and save intermediate changes, etc. etc.)

One important note is that the only thing you should put in github are
the updated files (Snakefile, in this case). All of the
_generated_ files can be reproduced from the repo and so you don't need
to add them.

**Please only commit changes to the Snakefile, and don't add any other files with `git add`.**

## 6. Relax in knowledge of a job well done.

Please do inspect the Snakefile on github via the Web interface to make sure it has all your changes!

## Questions? Problems?

If you run into any trouble with submission, that's ok - just let me know.

You can reach me via e-mail at ctbrown@ucdavis.edu, or in UC Davis slack under @ctitusbrown.
