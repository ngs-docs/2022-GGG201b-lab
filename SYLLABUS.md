---
tags: ggg, ggg2022, ggg201b
---

# Syllabus for GGG 201(b) lab section: Genomics (Winter 2021/2022)

[toc]

UC Davis

### GitHub site: [github.com/ngs-docs/2022-GGG201b-lab](https://github.com/ngs-docs/2022-GGG201b-lab)

This site will contain all of the actual lessons :).

## Code of Conduct

Please abide by [my lab's Code of Conduct](http://ivory.idyll.org/lab/coc.html) in this course.

In particular, this is not an intellectual contest, and please realize that we all have plenty of things to learn.

## Lab sessions

What: hands-on computational work

When: Friday 9-10:50am

Where: Zooooooom!

(I anticipate keeping the lab "virtual" for the entire quarter, by default.)

### Recordings

Lab sessions will be recorded and (privately) posted for all eternity; the links to lab sessions and any associated notes will be in the comments under the announcement for that lab.

### Computational resources

We'll be using cloud computers (lab 1) or the UC Davis High Performance Compute Cluster; all you'll need is an Internet-connected laptop.

## Homework

There will be three homeworks, submitted via GitHub Classroom.

Homeworks can be done collaboratively, but you need to hand it in individually.

## Grading and collaborative work

I grade each homework S/U. The division of grading between labs and
the rest of the course is in the whole course syllabus.

## Instructors

C. Titus Brown (IOR) (<ctbrown@ucdavis.edu>)

## Office hours

You can sign up for office hours using [this calendly link](https://calendly.com/ctitusbrown/30min), or e-mail Titus at ctbrown@ucdavis.edu.

## Lab description

In this lab, we will build and examine three different automated workflows, for three common bioinformatics tasks: variant calling, genome assembly, and RNAseq differential expression. (These will not be cutting edge workflows and should not be directly used for your own work, but they will be complete and functional.)

The overall learning goals for the lab are to:

1. familiarize you with the basic operational concepts involved in variant calling, genome assembly, and RNAseq differential expression.
2. introduce the use of workflow management tools as a core aspect of biological data analysis.
3. describe scientific issues surrounding data analysis techniques and processes, including statistical issues, reproducibility, provenance, and publication.

In terms of technology, we'll be using the snakemake workflow system, running on the farm cluster. We'll touch briefly on git/GitHub and conda, but those topics will be discussed in much more detail in [GGG 298, Tools for Data Intensive Research (winter 2022)](https://hackmd.io/PpgYFrY3SMuGsDsB6VMoMw?view)!

### Lab sessions and topics

Labs 1-4 - variant calling and snakemake workflows
Labs 5-7 - de novo genome assembly
Labs 8-10 - RNAseq for differential expression analysis

(The lab sessions build on each other and they do not track the lectures very well. This is kind of unavoidable, sorry.)

### Teaching style

The labs are taught in a very hands-on way, with an emphasis on running things first, and then going back and exploring what the commands are doing. That means that not everything is explained in detail the first, or even the second time, you see it! For the first few sessions, please bear with - hopefully it will all make sense by the end, and if not, ask questions!
