### Future discussions
1. How to create a DSL

### January 14 2021 ###
Topics for discussions:
1. How to allow variable size communications
1. How to do task batching

### September 24 2020 ###

No agenda yet.

### September 10 2020 ###

1. What is a gather control and how to use it.
1. Is there a way to disable parsec thread binding and use the --map-by ppr:1:socket within OpenMPI to make sure that all the worker threads in a process execute in the same socket?
1. Using a separate thread to push tasks to an idle process and request tasks when worker threads cannot find ready tasks is a reasonable approach. Does it make sense to merge this with the communication thread? Would it make senseto have this thread bound to the same core as the communication thread
1. Question about the Haar-tree example: The program is not terminating. Is there any specific threshold and alpha values for this test?

### July 16 2020 ###
1. What is prepare_input and how tasks inputs are manipulated in PTG ?
1. How to correctly release a task ? (even when migrated)
1. This is not a question but an outcome of the discussion: Can we clean the dependencies tracking before prepare_input ? The idea is that once a task is allocated (which means all input dependencies are available), we do not need the tracking support for the task anymore and we can eagerly release the hash_dep.

### June 18 2020 ###
1. Task migration
2. Task tracking
3. [Visualization Tools](https://bitbucket.org/icldistcomp/parsec/wiki/VisualizationTools.wiki)

### June 4 2020 ###
1. The ParSEC schedulers
    - How to select tasks that can be migrated
1. Hardware specific optimization mechanisms for DPLASMA that depends on ParSEC JDF.\\
    - autotuning
1. Mechanism behind shared memory layout optimization in parsec - how are decisions made for NUMA aware handling of data?

### May 21 2020 ###

**Topic**:  What is Modular Component Architecture (MCA) and how it applies to PaRSEC.

### May 07 2020 ###

**Topic**: PaRSEC's Profiling Infrastructure and capabilites

[Overview - Thomas Herault](https://bytebucket.org/icldistcomp/parsec/wiki/files/pug070520_prof_therault.pdf)

[The POTRF Use Case - Yu Pei](https://bytebucket.org/icldistcomp/parsec/wiki/files/pug070520_prof_yupei.pdf)

* [R playground (via Google Colab)](https://colab.research.google.com/drive/1a42_sNw9fAJafaI_RXHQXPEXnxbFNNcz?usp=sharing)

### April 23 2020 ###

**Topic**: Communication Engine

### April 9 2020 ###

**Topic**: Miscellaneous (data manipulation, data copies, data collections)

### March 26 2020 ###

**Topic**: What is PaRSEC again ?