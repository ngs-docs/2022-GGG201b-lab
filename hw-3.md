---
tags: ggg, ggg2022, ggg201b
---
# GGG 201(b), Lab Homework 3 - 2022

[![hackmd-github-sync-badge](https://hackmd.io/pnzNOH5KQKmKugAFbY7RKA/badge)](https://hackmd.io/pnzNOH5KQKmKugAFbY7RKA)

([Permanent link on github](https://github.com/ngs-docs/2022-GGG201b-lab/blob/main/hw-3.md))

Due by 10pm, Tuesday March 15th

For this homework, you're going to add some samples to [the RNAseq analysis pipeline we used in weeks 8 and 9](https://hackmd.io/447eXyGsT_OmP8ySC05tyA).
As you may recall, this RNAseq workflow conducted a differential expression analysis of three wild-type (wt) yeast data sets against three snf2 knockouts.

For HW #3, you'll add
add two new data sets into the workflow, and adjust the p-value cutoff to be 0.05 instead of 0.1. 

## 1. Start up the RNAseq binder

Start up a new binder:

[![Binder](https://aws-uswest2-binder.pangeo.io/badge_logo.svg)](https://aws-uswest2-binder.pangeo.io/v2/gh/nih-cfde/rnaseq-in-the-cloud/stable?urlpath=rstudio)

::::info
You can stop a running binder instance **[at your hub home](https://hub.aws-uswest2-binder.pangeo.io/hub/home)**, in case it tells you that you already have one running but you can't find the tab.
::::

## 2. Make edits etc.

In your running binder, do the following four steps:

(a) edit the Snakefile and add the links to the two new data sets to the SAMPLES dictionary at the top:

ERR458496 - URL https://osf.io/tzagu/download

ERR458503 - URL https://osf.io/px7sf/download

(You can see more about the data sets 
[here](https://www.ebi.ac.uk/ena/browser/view/ERR458496?show=reads) and [here](https://www.ebi.ac.uk/ena/browser/view/ERR458503?show=reads)).

(b) edit `rnaseq_samples.csv` to load the new sample `quant.sf` files that will be produced by the Snakefile; note that ERR458496 is `wt` and ERR458503 is `snf2`.

(c\) Edit line 26 of `rnaseq-workflow.Rmd` to set the p-value cutoff to be 0.05.

(d) Edit line 487 of the `rnaseq-workflow.Rmd` file to contain eight conditions (4 wt, 4 snf2).

Next, run the snakemake workflow with
```
snakemake -j 4 --use-conda
```
and then load `rnaseq-workflow.Rmd` and knit it to HTML.

Verify that your HTML workflow report shows the new data sets in the PCA and the MDS plots, and that the individual gene report has four points for each condition.

## 4. Save your files to your local computer

You'll have four changed files to preserve and hand in. They are:

* `Snakefile`
* `rnaseq_samples.csv`
* `rnaseq-workflow.Rmd`
* `rnaseq-workflow.html`

You can download these to your laptop by selecting them in the file browser and selecting More... Export.

## 5. Upload the four files to github.

Accept the [HW 3 assignment](https://classroom.github.com/a/tWSqU_vu), and once your hw3 repository is created, use the "Add file..." "Upload" menu to upload all four files. You can commit directly to the stable branch (i.e. follow the defaults).

You can check under "Pull requests", "Feedback", "Files changed" to see if the three text files (Snakefile, rnaseq_samples.csv, and rnaseq-workflow.Rmd) have the right changes (a-d, above).

And then... you're done! Congrats!
