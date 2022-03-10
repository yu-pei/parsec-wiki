### Future discussions
1. How to create a DSL

### March 10 2022 ###
The main topic of discussion is the support for accelerators, how different types of tasks map onto them, and how the memory management is handled.

Here is a more detailed list of topics:

1. What is recursive in devices?
2. If there are 2 GPUs, what is their naming scheme
  - dev_0 – CPU
  - dev_1 - recursive
  - dev_2 - GPU1
  - dev_3 - GPU2?
3. How is memory allocated in GPU?
  - Is 95% memory initially allocated and then managed separately using
  - Zone_malloc_init()
  - Zone_malloc()
  - Is task->data[i].data_out used to hold the GPU data?
4. Can we use CUDA occupancy for load calculation?
  - NVIDIA Management Library (NVML)
  - nvmlDeviceGetUtilizationRates()
  - Are all tasks executed by a single CUDA thread?
5. What do these signify in a CUDA body?
  - `BODY [type=CUDA dyld=cublasDgemm dyldtype=cublas_dgemm_t weight=(1)]`


### January 14 2021 ###
Topics for discussions:

1. How to allow variable size communications
1. How to do task batching

Slides: [Local iterators in PTG](https://bytebucket.org/icldistcomp/parsec/wiki/files/pug011421_ptg_local_iterators.pdf)

[Block sparse GEMM Repository](https://bitbucket.org/herault/irr-gemm-gpu-over-parsec/src/master/)

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