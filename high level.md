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