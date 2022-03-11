---
tags: ggg, ggg2021, ggg201b
---
# Lab 10 outline - more RNAseq; workflow systems - GGG 201(b) lab, March 11th, 2022

[![hackmd-github-sync-badge](https://hackmd.io/zpk5b79BSvaSBns4HUPD8g/badge)](https://hackmd.io/zpk5b79BSvaSBns4HUPD8g)


[toc]

## RNAseq

### Revisiting our RNAseq workflow.

Start up a new binder:

[![Binder](https://aws-uswest2-binder.pangeo.io/badge_logo.svg)](https://aws-uswest2-binder.pangeo.io/v2/gh/nih-cfde/rnaseq-in-the-cloud/stable?urlpath=rstudio)

::::info
You can stop a running binder instance **[at your hub home](https://hub.aws-uswest2-binder.pangeo.io/hub/home)**, in case it tells you that you already have one running but you can't find the tab.
::::

### Run things

Terminal,
```
snakemake -j 4 --use-conda
```
and then open `rnaseq-workflow.Rmd`, knit, HTML.

### Explore the various config files

`Snakefile` - what produces the `quant.sf` files.

`rnaseq_samples.csv` - used to configure `rnaseq-workflow.Rmd`

p-value cutoff at top of `rnaseq-workflow.Rmd`.

Pro-tips:
* You only need to rerun snakemake from scratch when you add samples, and conveniently snakemake will figure that out for you.
* knitting the HTML document will regenerate all of the outputs.

### p-values, q-values, and FDR

What's the difference between a p-value and a q-value?

The latter controls for the *number of comparisons* you're doing.

Analogy:
* if I asked you what the likelihood of flipping three fair coins and getting three HEADS was, what would you say?
* what if I let you flip three coins, twice? And still only asked for one set of three HEADS among the two trials?
* How about 5 times? 10 times? At what point would your expectation for the number of times you got three HEADS exceed 1?

See also: p-hacking. https://projects.fivethirtyeight.com/p-hacking/

### Picking thresholds - which genes are "significant" and worthy of follow-up?

The statistical approached used by DESeq2 properly corrects for all sorts of things, including variability in expression levels on a *per gene* basis.

The thresholds for q-values estimate how many of your results (under said statistical model) are likely to be "false". Basically you get to choose between sensitivity (but more false positives - genes that aren't actually diff expr) and specificity (but more false negatives - things you missed).

Pick a threshold and stick with it. Do not pay  attention to the ratio of expression levels.

tl;dr - unless you have a super-strong biological reason to believe that RNA abundance levels are important, **pay attention to the q-values, not the expression ratio.**

(hint, you probably do not have a super-strong biological reason.)

## Some thoughts and perspectives on this lab section

Software changes. You will be using different programs in three years.

Data types change. You will be working with a dizzying variety of data types, and many of them will not have existed in their current form more than three years ago.

Compute systems change. You'll probably be using different computers than farm for your work.

I've tried to walk a middle ground between:
1. teaching you something actually useful. (You can use the software I've shown you for real biology.)
2. being able to do stuff hands on. (You _have_ to see things running and look at actual output files if you want to understand stuff.)
3. not overwhelming you with information.
4. introducing you to skills and perspectives that you can grow into.

I think this year has had the best and most complete technical content I've delivered, but remote kinda sucked (more than usual). Thinking about how I might do things differently next year.

Suggestions welcome - e-mail OK, but I'm happier to do via zoom chat or coffee/tea/boba (I'll buy)

### Workflows are the future

...that's why I'm teaching them to you.

For inspiration, read [Mick Watson on three technologies that bioinformaticians need to be using right now](http://www.opiniomics.org/the-three-technologies-bioinformaticians-need-to-be-using-right-now/).

### Why use workflow systems? The boring technical reasons.

---

A list of specific reasons -

- declarative vs procedural specification
  - allows analysis of workflow graph, parallelization
  - allows declaration of specific software needed, for later tracking
  - can compare workflows more easily, in theory (=> reproducibility)
  - can rerun failed steps
- tracks dependencies on files
  - allows rerunning just the bits that need to rerun
      - (this is more important than you know :)
- reusability, in theory
- different conda/docker software environments for each step
  - allows pinning of software versions
  - allows use of potentially incompatible software
- cloud compatibility/HPC portability

---

### The first _real_ reason: lots of steps.

Bioinformatics workflows involve a lot of steps and you need to do it on a lot of samples. You don't have _time_ to type all that stuff out.

Look at our RNAseq workflow... how many steps are involved!? For each sample, you are doing at least four steps! Multiply by 100 samples...

### The second _real_ reason: data integration.

99.9% or more of the data you will use in your research is not your own data!
- data integration with databases, published work
- raw data reanalysis and reinterpretation

this involves complex computational pipelines that produce a bunch of results and cross-check and compare and ...

### Workflow systems make this _possible_.

...not necessarily easy, but _possible_.

how to get started doing technical stuff you don't know how to do already: the 5 stages
1. be aware of the possibility
2. start doing it (you're already pretty far into it)
3. keep doing it
4. ask for help and use google a lot
5. realize you're more expert than anyone you work with

### Some references

Principles for data analysis workflows
https://journals.plos.org/ploscompbiol/article?id=10.1371/journal.pcbi.1008770
This is a _conceptual_ article on how to do data analysis.

Workflow systems turn raw data into scientific knowledge
https://www.nature.com/articles/d41586-019-02619-z
Nature Toolbox blurb on workflow systems,

Executable manuscripts
https://www.nature.com/articles/d41586-022-00563-z
Nature Toolbox blurb on how to write your thesis better

Reproducible, scalable, and shareable analysis pipelines with bioinformatics workflow managers
https://www.nature.com/articles/s41592-021-01254-9
Technical perspective.

Streamlining data-intensive biology with workflow systems
https://academic.oup.com/gigascience/article/10/1/giaa140/6092773?login=true
Technical perspective.

Perspectives on automated composition of workflows in the life sciences 
https://f1000research.com/articles/10-897/v1
Technical/forward-looking perspective.

## Next steps

One thing you're missing at this point is _access to compute_. I'm working on that...

Another thing you might be missing at this point is _access to RStudio on big computers_. I'm also working on that. See [Lab 10 of GGG 298](https://hackmd.io/SJxLcVlTTRuY43gB7j_CIw?view) for a direction we're pursuing for running RStudio on farm, for example.

The last thing you might be missing at this point is a community of people interested in data-intensive stuff. A few pointers -
* the UC Davis DataLab is a hub for Data Science activity on campus. (I'm working with them closely on compute access and training.) I'll try to make sure our activities make it out to y'all, but a good place to start is to sign up for [datascience-announce](https://datalab.ucdavis.edu/mailing-list/).
    * You might be interested in the new [Grad Pathways in Research Computing](https://gradpathways.ucdavis.edu/research-computing-pathway). You're already pretty far along, hint. (That is not entirely coincidental :)
    * You might find a lot of these DataLab-associated materials ...interesting. https://ngs-docs.github.io/2021-august-remote-computing/
* you might be interested in the [Davis R Users Group](http://d-rug.github.io/). There is no friendlier bunch of folk on campus!
* there's a lot of centers and collaborations and groups on campus - [AI in Food Systems](https://aifs.ucdavis.edu/), [Center for Data Science and Artificial Intelligence Research](https://cedar.ucdavis.edu/), [AI in VetMed](https://ai.vetmed.ucdavis.edu/), etc.

## Closure

Good-bye and good luck!

I'm always happy to chat; book me on [calendly](https://calendly.com/ctitusbrown/).
