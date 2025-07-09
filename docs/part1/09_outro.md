# 1.9 Wrapping up

Congratulations on completing Part 1 of the Nextflow for the Life Sciences workshop!

Throughout each lesson, you've built up a fully functional, and flexible Nextflow workflow from scratch. Along the way, you've gained practical experience with the key concepts and syntax that form the backbone of Nextflow pipelines.

## 1.9.1 Key takeaways

* Nextflow lets you to chain together data processing steps that allow you to make complex bioinformatics workflows
* Nextflow workflows are built from small units (processes) that communicate
via channels
* A variety of execution logs and outputs lets you troubleshoot your workflows
* Work and task directories are uniquely identified to enable resumability and checkpointing
* Channels are a powerful feature that lets you structure your workflows and provides the ability to run multiple tasks in parallel
* Using parameters can make your pipeline flexible and configurable upon runtime - no more hardcoding and editing your files everytime new data or samples are run

## 1.9.2 What's next?

In Part 1 we developed a simple "Hello World!" example to introduce the key Nextflow
concepts required to build more complex, real-world bioinformatics workflows.

Part 2 will continue by taking the concepts and examples introduced here and
apply them to develop a simple RNAseq processing pipeline.
