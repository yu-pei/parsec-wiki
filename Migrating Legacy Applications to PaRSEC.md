# Migrating Legacy Applications to PaRSEC

This document presents an overview on how to migrate legacy
applications over the PaRSEC runtime. It complements the information
accessible in published papers
[1](https://ieeexplore.ieee.org/document/7101660/),
[2](https://ieeexplore.ieee.org/document/7307597/),
[3](https://dl.acm.org/citation.cfm?id=3148233), and the API
documentation available inside the PaRSEC source code. 

##Basics: Initializing PaRSEC

PaRSEC must be initialized (resp. finalized) after MPI (resp. before
MPI). The initialization creates a pool of threads that remain silent
until a PaRSEC progress function is entered. How many threads are
created, and other behaviors of the PaRSEC runtime are controlled by
the command line arguments provided to ```parsec_init``` and by the
contents of the MCA configuration file of the end user (see
[Using PaRSEC binding to bind computation and communication cores](Using PaRSEC binding to bind computation and communication cores.md) and
[How to write a program that uses a PaRSEC enabled operation](datadistribution.wiki)).

```
#!c

#include "parsec.h"

<...>

int main(int argc, char *argv[])
{
  <...>
  parsec_context_t *parsec_ctx;
  
  MPI_Init(&argc, &argv);
  parsec_ctx  = parsec_init(-1, &argc, &argv);

  <...>

  parsec_fini(&parsec);

  MPI_Finalize();
  return 0;
  }
```

## Basics: Compiling and linking with PaRSEC

PaRSEC comes with two techniques to help integration in a larger
project: pkg-config and CMake.

Using pkg-config, the user will find the include and link options to pass to
their compiler using the following command lines (other pkg-config
specific flags are also available, see the manpage of pkg-config).

```
#!sh

pkg-config --cflags parsec
pkg-config --libs parsec
```

To use CMake, the user will find a ```FindPARSEC.cmake``` module
installed in the PaRSEC installation directory, under the subdirectory
```lib/cmake/FindPARSEC.cmake```. Once CMake is configure to point to
the appropriate directory, the user needs to issue a
```find_package(PARSEC)``` which defines ```PARSEC_FOUND``` if PaRSEC
is found, and in that case also defines ```PARSEC_INCLUDE_DIRS,
PARSEC_LIBRARIES```, and other variables (see the header of
```FindPARSEC.cmake``` for further documentation).

## Integrating the application with PaRSEC

Users are not required to port all routines using the PaRSEC task
system. PaRSEC is designed to work with other paradigms, and multiple
integration schemes are possible. We describe here a few that are
common.

### Using a PaRSEC-enabled operation

The first approach is to use PaRSEC as a black box, relying on other
operations that have been designed over PaRSEC as any other
libraries. The wiki documentation
[How to write a program that uses a PaRSEC enabled operation]([datadistribution.wiki)
covers this information. Users are encouraged to read this part, as it
also explains how the data distribution are managed, which is a
necessary information to enable a tighter integration.

### Porting some operation into a PaRSEC-enabled operation

As PaRSEC is a multi-Domain Specific Languages runtime, there are as
many options as DSLs supported by PaRSEC to port a user-defined
operation over the PaRSEC runtime. The two main approaches are
Parameterized Task Graphs and Dynamic Tasks Discovery.

Parameterized Task Graphs (PTG) are most suitable when the programmer
has a clear understanding of the entire dataflow, and can express the
dependencies between tasks analytically. PTGs provide a high-level
representation of the entire DAG of tasks, and allow the PaRSEC
runtime to manage efficiently extreme numbers of tasks. How a PTG is
written is described in
[How to write a PaRSEC enabled operation by creating a JDF file](writejdf.wiki)
(Job Data Flow, or JDF, is the file format used by PaRSEC to express a
PTG).

Dynamic Tasks Discovery (DTD) is the approach of choice to build a
prototype (as the development time of a DTD DAG is often lower than
the time required to write a PTG), or when the DAG is highly
irregular. In its simplest form, DTD relies on a sequential thread
that iterates over the tasks, and exposes how  each task modifies or
accesses data. Transparently for the user, the runtime system builds a
DAG of tasks that exposes the maximum parallelism, while still
preserving the same semantics as the one defined by the sequential
execution of tasks, as discovered by the iterating thread. That DAG is
executed in parallel and in the background, as the sequential thread
iterates over the tasks, and until all discovered tasks are completed.

We will describe in further details below how to define an operation
using the DTD paradigm. We will use to do so the ping-pong example.

### Porting the ping-pong operation into a PaRSEC-enabled operation using the DTD paradigm

The first thing a programmer needs to do to port an operation into a
PaRSEC-enabled operation is to define the tasks and their
granularity. Tasks in PaRSEC are 'pure' functions with well-defined
(and described) inputs and outputs (a given input can also be used as
output). Usually, they are sequential code that read only the elements
of data described to the runtime system (the inputs), and modify only
the elements of data described to the runtime system (the outputs).

>In our example, a task will either be a 'ping' or a 'pong'
>function. The Ping function initiates some data, that the Pong
>receives, doubles, and sends back to Ping.

Tasks belong to task classes. Task classes are used to group tasks
under a similar operation (e.g., if the target operation uses many
tasks that apply the same function G on many data elements: 0, 1,
... N, G will be a task class, while G(0), G(1), ... G(N) will be
tasks). At runtime, tasks also belong to a taskpool. A taskpool
represents the DAG of tasks as it is known at a given time by the
runtime system.

> In our example, we will initiate the DAG with a Ping(0) task that
> will send a data to Pong(0), multiply it by 2 and send it back to
> Ping(1), that adds 1 to it. The DAG iterates until Pong(N). Ping
> and Pong are task classes, Ping(i) and Pong(i) are tasks, and they
> will all belong to the ```ping_pong_tp``` taskpool. We first
> create the taskpool:
> 
> ```
> #!c
> 
> parsec_taskpool_t *ping_pong_tp = parsec_dtd_taskpool_new(  );
> parsec_context_add_taskpool( parsec, dtd_tp );
> parsec_context_start(parsec);
> ```
> As it is a DTD taskpool, we use the ```parsec_dtd_taskpool_new``` to 
> create it. The taskpool is then added to the PaRSEC context, making it
> accessible to run on these resources, and it is started, so any task
> discovered whose input dependencies are ready can start executing right
> away.
>
> Note that this operation is done by all MPI processes. In the simple DTD
> paradigm, all MPI processes will discover the same entire DAG. The
> data distribution, and task affinity (see below), are used to
> distribute the work between the processes, and strategies to prune
> the DAG to a minimal are implemented internally. All tasks must be
> discovered by all MPI processes, however, for the DTD engine to
> build a correct DAG of tasks.

The second thing a programmer needs to do to port an operation into a
PaRSEC-enabled operation is to describe the data onto which the tasks
will operate. This is done through the Arena concept in PaRSEC. Arenas
are used to allocate temporary data when necessary, and to transmit
data from one node to another. In order to do so, they need to know
the shape of the data elements (i.e. their size, offsets, location,
etc.). This information is transmitted to PaRSEC by defining arenas,
one per data element shape that is used by a given taskpool. For
example, if the user has two task classes that take as input a vector
of double and a matrix of floats, an arena for the vector of doubles
and another arena for the matrix of floats must be defined for this
taskpool.

> In our example, Ping tasks will send a vector of 2 doubles to Pong tasks, 
> that will answer with the same shape of data, so a single arena needs to
> be defined. In the DTD paradigm, that arena is added to the 
> ```parsec_dtd_arneas``` array:
> 
> ```
> #!c
> 
> parsec_matrix_add2arena_rect(parsec_dtd_arenas[TILE_FULL],
>                              parsec_datatype_double_t,
>                              2, 1, 2);
>```
> We use ```parsec_matrix_add2arena_rect```, which is a helper function
> defined in ```parsec/data_dist/matrix/matrix.h```. It creates a matrix of
> 2x1 consecutive doubles. Arbitrary data types can be defined using the MPI
> data type engine.

Next, the programmer needs to allocate data for the tasks to work on,
and distribute it among the MPI processes. This defines a data
collection for PaRSEC. PaRSEC comes with a large library of data
collections. [How to write a program that uses a PaRSEC enabled operation](datadistribution.wiki)
covers how to define additional data collections.

> In our example, there will be only 2 data elements: one stored on rank 0, 
> and the other stored on rank 1. We can use a 2D Block-cyclic distribution 
> to allocate and distribute the data:
> 
> ```
> #!c
> 
> two_dim_block_cyclic_t *dcA = malloc(sizeof(two_dim_block_cyclic_t));
> parsec_data_collection_t *dA = (parsec_data_collection_t)dcA;
> two_dim_block_cyclic_init(dcA, matrix_RealDouble, matrix_Tile,
>                           worldsize, rank,
>                           2, 1,
>                           2*2, 1,
>                           0, 0,
>                           2*2, 1,
>                           1, 1,
>                           MPI_COMM_WORLD);
>
> m->mat = parsec_data_allocate((size_t)m->super.nb_local_tiles *
>                               (size_t)m->super.bsiz *
>                               sizeof(double));
>```
> ```two_dim_block_cyclic_init``` takes many parameters, to describe the
> size, stride and location of each block of data. It is fully documented in 
> the manual of PaRSEC. To simplify the explanation, this creates a "matrix"
> of 4 doubles by 1 double, distributed between processes 0 and 1, each
> getting a block of 2 by 1. Once the data has been described, the
> field 'mat' of the ```two_dim_block_cyclic``` type must point to the
> local data, which must be contiguous (each local block of data is located
> after the other). The ```parsec_data_allocate``` function ensures that
> the data is properly aligned to allow pinning and efficient transfers
> with the network cards and the accelerators if there are some. 

Once the data has been allocated, distributed and described to PaRSEC, it
must be exposed to the DTD paradigm.

> ```
> #!C
> 
>  parsec_data_collection_set_key(dA, "A");
>  parsec_dtd_data_collection_init(dA);
> ```

A task class is associated to a body. A body is a native-language
function that executes some code on the data. The runtime system is
responsible of providing the input data to the body, and dispatching
the output data to other functions, following the DAG partial order of
tasks. The body contains the user code that the programmer wants to
see applied on data elements.

In DTD, a body takes only two arguments: the ```execution_stream``` and 
the task itself. An execution stream in PaRSEC represents a computing
resource (e.g. a core). The runtime system uses this to store thread-local
information and some PaRSEC functions (e.g. function to allocate temporary
memory) use it to avoid atomic operations.

The task is another opaque object that stores information. In DTD, part
of the information stored in the task is the data and parameters that
were managed by the runtime system or passed during task discovery. The
programmer can access these informations using ```parsec_dtd_unpack_args```.

> In the context of the example, we need two bodies: one for the ping tasks,
> and one for the pong tasks. A body is defined as follows:
> 
> ```
> #!c
> int ping( parsec_execution_stream_t  *es,
>           parsec_task_t *this_task )
> {
>     (void)es;
>     int task_id;
>     double *data_in, *data_out;
> 
>     parsec_dtd_unpack_args(this_task, &task_id, &data_in, &data_out);
>     if( task_id == 0 ) {
>       data_out[0] = 1.0;
>       data_out[1] = 1.0;
>     } else {
>       data_out[0] = data_in[0] + 1.0;
>       data_out[1] = data_in[1] + 1.0;
>     }
> 
>     return PARSEC_HOOK_RETURN_DONE;
> }
>
> int pong( parsec_execution_stream_t  *es,
>           parsec_task_t *this_task )
> {
>     (void)es;
>     int task_id;
>     double *data_in;
>     double *data_out;
> 
>     parsec_dtd_unpack_args(this_task, &task_id, &data_in, &data_out);
>     data_out[0] = 2.0 * data_in[0];
>     data_out[1] = 2.0 * data_in[1];
> 
>     return PARSEC_HOOK_RETURN_DONE;
> }
>```
> Ping checks the task identifier (which is a parameter that the programmer
> will pass to the task when discovering it), and if it is 0, it initializes
> the data to 1.0. Otherwise, there is nothing to do to the data in this
> case. Pong gets the task identifier and the data, and doubles each element
> of data.

The bodies are defined symmetrically to the way they are used. The programmer
uses them by 'discovering' the tasks in a sequential order, and inserting
them into the taskpool.

>```
>#!c
>
> for(int i = 0; i < N; i++) {
>   parsec_dtd_taskpool_insert_task(ping_pong_tp, ping, 0, "ping",
>        sizeof(int),    &i,                VALUE,
>        PASSED_BY_REF,  TILE_OF_KEY(dA, 0), INOUT | TILE_FULL | AFFINITY,
>        PASSED_BY_REF,  TILE_OF_KEY(dA, 1), IN    | TILE_FULL,
>        PARSEC_DTD_ARG_END);
>   parsec_dtd_taskpool_insert_task(ping_pong_tp, pong, 0, "pong",
>        sizeof(int),    &i,  VALUE,
>        PASSED_BY_REF,  TILE_OF_KEY(dA, 0), IN    | TILE_FULL,
>        PASSED_BY_REF,  TILE_OF_KEY(dA, 1), INOUT | TILE_FULL | AFFINITY,
>        PARSEC_DTD_ARG_END);
> }
>```
> 
> ```parsec_dtd_taskpool_insert_task``` is a variadic argument function
> that discovers a task and inserts it into the corresponding taskpool.
> It also connects the body to the task, and to the data that the task
> reads or modifies.
> 
> In the example, ping and pong both take three arguments: the first one is a
> parameter, passed by value (i), the second is a data that the body will
> read, and the third one, the data it will modify.
> VALUE parameters are specified by their size and address; data elements
> define the arena they belong to (```TILE_FULL```), how they are accessed
> (```INOUT``` or ```IN```), which one they are (```TILE_OF_KEY(dA, 0)```),
> and if the task should execute where the data is located (```AFFINITY```).
 
Finally, the programmer must specify when no data will be modified by a DAG.
This condition is necessary to detect that all the work has been discovered
for this taskpool. Once this is done, the thread that was used to discover
the tasks and build the DAG can join the computation with the other threads,
and wait for the completion of the taskpool.

> ```
> #!c
> 
>     parsec_dtd_data_flush_all( ping_pong_tp, dA );
>     parsec_context_wait(parsec_ctx);
> ```

Once the function ```parsec_context_wait``` completes, all tasks that
were discovered and inserted in the ```ping_pong_tp``` task pool are
completed, and the data that was exposed to the DAG contains the
result of the operation. All that remain is to cleanup the resource
that was allocated.

> ```
> #!c
> 
> parsec_arena_destruct(parsec_dtd_arenas[TILE_FULL]);
> parsec_dtd_data_collection_fini( dA );
> free_data(dcA);
> parsec_taskpool_free( ping_pong_tp );
> ```
