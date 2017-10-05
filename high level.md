##  Modular Architecture

PaRSEC is designed following a Modular Component Architecture that enables final users and application developers to select specific features at run time to adapt the task system to distinct behaviors.  As an example, the node-level task scheduler is provided as a component, which allows users to select dynamically what implementation of the scheduler is used for the current set of tasks. The runtime system comes with a predefined set of schedulers (favoring cache reuse for NUMA machines, or the user-defined priority, or many flavors between the two), but through the use of dynamic library loaders, the user may provide her own scheduler, following the API of the module.

## Profiling and Instrumentation

Another example of modular component used in PaRSEC is the PaRSEC Instrumentation Module, built on top of the Profiling system. PaRSEC allows end users to obtain detailed information on the execution of tasks, either for post-mortem analysis or during the execution to tune parameters. As tasks get scheduled on computing resources, enabling more tasks, and progressing the computation, they follow a finite state machine description of their life cycle. In its simplest form, a task gets instantiated by the release of its input flows, then the data corresponding to each flow is pulled from their current repository, the task gets executed by a core or an accelerator, releasing new data, and the task is the released. The state machine of PaRSEC allows the application developer and the runtime system to create additional transitions, allowing a task to return in the data pulling state, if the code of the task requires to move on a different device, for example. Attached to each transition of this finite state machine, a tracing system records the times of each operation, and can, at the user request, decorate these records with user-selected hardware counters or user-defined parameters. To simplify the selection of these events, a selection of modules that can be loaded at runtime are made available. Similarly to the scheduling capabilities, users may define their own and provide them at runtime by following the API defined in the modules.

## Communication Engine

PaRSEC is a distributed task system for Exascale applications. The runtime system manages the communication transparently for the application developer: following a data flow approach, the algorithm is expressed as a set of tasks applied on data. Data elements are described by the application developer for the runtime system, and starting from that point, the runtime system moves the data where the tasks require them, without the need for the application to explicitly call MPI routines. Communications happen in the background, in order to overlap communication between each other and with computations, hiding the cost of message passing, both from a programmatic and a performance point of view. Communications are scheduled following an eager strategy that aims at getting the data ready as soon as possible without hindering the computation progress.

## Programming Interfaces

PaRSEC features multiple programming interfaces, and is designed to be extended to additional interfaces that are fitted for the scientific domain of the application developer. The two general purpose programming interfaces distributed with the main PaRSEC library are Parameterized Task Graph (PTG) and Sequential Task Flow (STF).

Sequential Task Flow is pervasive in modern task systems: a thread inspects the set of tasks to execute, discovering them as it unrolls a sequential algorithm. Instead of executing each task as it is discovered, that task is saved in an internal representation of the Directed Acyclic Graph of Tasks, each node of that graph representing the discovered task. The API of the STF allows the application developer to express what data each task accesses, with what particular traits. Each incoming edge of that node then represent the dependences to other tasks previously discovered. These dependences are inferred from the Bernstein conditions over the data accessed by each task. As the DAG is discovered, tasks that become ready get scheduled on cores or accelerators by the variety of schedulers available in PaRSEC. The execution, although it is parallel, is guaranteed to be semantically equivalent to the execution of the serial code, providing an elegant interface for automatic parallelization.

Parameterized Task Graph is less common. A PTG is a synthetic and abstract representation of the entire directed acyclic graph of tasks. Leveraging on the parameterization, the representation is compact and does not depend on the problem size. Although this Interface demands from the application developer a deeper understanding of the dataflow of her algorithm, the benefits of improved scalability and reduced management overheads make PTG the Interface of choice for the DPLASMA library that is built on top of PaRSEC.

## Data Management, Heterogeneous Architectures, and Interface with Other Paradigms

As a dataflow engine, PaRSEC manages the data that is passed to the task system from their input in the Direct Acyclic Graph of Tasks to their output. Multiple versions of the same data may coexist at a given time, replicated on multiple nodes or memory devices, or different versions of the same data may be created to increase the parallelism and capacity of overlap. All the data movement is implicit in the algorithm description, that takes a declarative approach: tasks announce what data they need, and what data they produce (or modify), and the runtime system is responsible to move or copy the required data in a bank accessible to the device on which the task is scheduled.

This naturally enables execution over heterogeneous architectures for a very small coding price. If a version of the task suitable to run on an accelerator is present, the scheduler may decide to execute that task on the accelerator, and the data management system will be responsible to acquire the appropriate copy of input and output data to enable that execution. From a programming point of view, the data are located on the accelerator memory and passed to the user-provided functions as pointers, without the need for the developer to implement the data movement. Data is also moved transparently between devices, while cores are busy computing ready tasks.

As not all parts of the application need to be written using the PaRSEC Interfaces, different levels of composition (coarse or fine) are available to expose the data consumed by a PaRSEC algorithm as they are produced, or to extract the data produced by a PaRSEC algorithm and provide them to another programming paradigm. This approach, that was tested for complex applications like NWChem, allows to transition from an imperative parallel programming approach to a dynamic task system approach step by step, without requiring the entire rewrite of a large set of the application.


OVERVIEW
========

PaRSEC (Parallel Runtime Scheduler and Execution Controller) is an
environment that proposes to simplify the design and accelerate the
execution of High Performance applications on leadership systems. PaRSEC
helps the application programmer in expressing parallelism without requiring
the intricacies of explicit parallelism programming models. In PaRSEC,
algorithms are expressed as a set of elementary tasks, which can then
execute efficiently at a massively parallel scale, as well as being
automatically delegated to accelerators. These tasks operate on explicit 
input data and produce (possibly in-place) output data, and the PaRSEC
runtime considers that data flow for 1) ensuring that tasks using the same
data execute in a correct order, and 2) automating data movement across 
node and between accelerators and hosts. Unlike most dataflow systems, PaRSEC
is designed from the get-go to execute on massively parallel and heterogeneous
(i.e. accelerated) systems, while at the same time being able to execute 
fine-grain tasks.

PROGRAMMING PARSEC
==================

PaRSEC is interoperable with legacy technologies (e.g. MPI, CUDA, etc.) which 
permits the incremental upgrade existing applications. Typically, a 
programmer will start from an existing SPMD application, and transform 
some particularly costly computational routine with PaRSEC, while leaving the 
rest of the application unchanged. The PaRSEC accelerated routine will 
typically look like and behave like an SPMD routine when viewed from the 
outside, while internally, it will unleash the full parallelism permissible 
from the dataflow. 

A PaRSEC routine is a set of task classes, representing elementary operations 
applied to blocks of data. Each task class describe the inputs, the output,
and the operator applied to the input data to produce the output data. Tasks
are pure and read or modify only input and output data. During the execution
all task instances are scheduled on the resources in parallel. 

The PaRSEC environment provides multiple front-ends, or **Domain Specific 
Languages** (DSL) tailored for the particular use case (chemistry, CFD, 
linear algebra, etc.) to write these task classes. High level DSLs input 
either C, C++, Fortran, depending upon 
the customs and preferences of the particular users community, and can 
produce the dependency between tasks automatically. The low level DSLs 
permit expressing task classes and their data dependency explicitly in 
a C-like language. PaRSEC tasks operate on data blocks, which can be sparse,
irregular, or have a datatype to represent complex memory layout. Data blocks
are assembled in distributed collections, which represent the distribution 
of the dataset on the machine. PaRSEC provides a rich toolbox of data 
collections to distribute data according to common or fully customized 
patterns, as well as common parallel operators (like maps, broadcasts, 
reductions) that operate on such collections. 

MACHINE ABSTRACTION
===================

The PaRSEC Runtime presents the hardware resources through an abstract 
machine comprised of multiple nodes. Each node contains multiple streams 
of execution, that each execute tasks serially. Streams can access any 
data in the node-local memory, so that the scheduler may select the most 
appropriate stream to execute ready tasks while optimizing at the same 
time data-reuse, NUMA access cost, and load balancing between streams. 
The end-user can specify how many streams will be 
used in a PaRSEC machine, their placement on physical cores, and may 
specify that a stream spans multiple cores (e.g. when the task kernel 
creates an OpenMP region). 

A node may also feature accelerator resources. An accelerator is represented
by a supplementary set of execution streams. For each task class, 
the programmer provides the CPU stream operator, and can optionally provide 
an accelerator operator (e.g. a CUDA function) as well as hints regarding 
the computational load of the operator. The PaRSEC runtime automatically 
copies the data between the host memory and the accelerators. Versioned data 
copies remain cached in the accelerator memory; coherency and version management 
of copies, as well as their transfer back-and-forth accelerators is handled 
by the PaRSEC runtime. Similarly, the selection of a CPU execution stream 
or a GPU execution stream is delegated to the scheduler.

The data collection describes the distribution of data blocks across multiple 
nodes. The end-user is in charge of selecting both the data distribution of 
the collections, and the affinity of tasks with nodes. Provided with this
information and the dataflow unfolding from the node-local scheduler, the 
runtime schedules asynchronously the necessary data transfer to move inputs 
to ready tasks execution sites. All communication operation are implicit 
in the program, yet the communication volume can be finely controlled by 
an expert programmer by setting the data collection distribution and 
task affinity.

RESULTS
=======

Some impressive results have been achieved using PaRSEC. PaRSEC greatly
simplify writing optimized linear algebra operations. In the Hierarchical QR
factorization (fig. x), the flexibility of PaRSEC permits expressing an
algorithm-tailored reduction tree that reduces the communication delay in
the performance critical panel, yet operates on a flexible distribution of
the data, and can thereby adapt to a variety of architectures and problem
sizes. This is an important feature of the dataflow programming approach in
which the depiction of the algorithm is independent of the data distribution
and resource mapping, thereby improving performance portability of optimized
codes.

We also have some chemistry doing water molecules and stuff, cool stuff
really.