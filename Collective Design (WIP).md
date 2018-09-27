This is the initial draft of the data-driven collective design in PaRSEC, that describe how collectives can be described and implemented in the runtime system and how they can be exposed to DSLs.

The goals for the reduction are:
* Different datatype t_1, t_2,... t_n sharing the same _typemap_ can be reduced into a final type t_0, type that obviously share the same _typemap_. As a possible extension, we can imagine a map-reduce type of operation, where the original _typemap_ can be dynamically changed during the initial stage of the collective.
* Processors in charge of doing the reduction are not necessarily the ones producing the data.
* Depending on the properties of the operator (commutative and/or associative) the order in which the operation is applied between the data (d_i) can be altered or not. If the order is altered then all local operations should be done first, so that each process minimize its participation to the global exchanges necessary for the internode part of the collective. Moreover, the knowledge about the architecture extracted from HWLOC, should be used to imilarly to what is going on in EVOLVE.

Challenges:
* Number of contribution is unknown and has to be specified to the DSL for termination detection.
* We might need the neutral element for the operation.

![ReductionDesign.png](https://bitbucket.org/repo/aX9b8L/images/666644334-ReductionDesign.png "Reduction in PTG")

> T(k)

Collective operation have an id `k` that has nothing to do with their execution space, but serve as a global key.

The operation has a set of mandatory properties, and optional hints that will help the runtime optimize the operation.

> [ op = reduce

The operation is type in this DSL to let the source-to-source compiler generate the proper functions.

> count = N

`count` is mandatory, this is the number of data that will flow through that operation globally.

> local = %{  ...  %}] 

On the other hand, `local` is an optimization. It gives the number of data that will flow through that rank only. Knowing when the local operation is done enable the runtime to start the final operation over the network as soon as possible.

The alternative of not knowing the local number of contributions forces the runtime to atomicly increment a counter on one node of the context to detect when the `count` number of contributions is achieved.

> : data(range)

This is a set of the processors allowed to execute task T. It does not mean that those processors are producers of the data to reduce.

> R    Left  <- B F(i,j)

This flow is implicit and must not be provided by the end developper. It will be inferred from the flows of task F.

> RW   Right <-  
>            ->  A G(0)     [type 2]

User will provide the reentrant flow, RW Right. It must have an output flow, it may define an input, this is still .

> BODY [ *PU ]

In the body, two flows are available, Left and Right. They are in "packed" format, it's the developer's job to implement the operation in packed format. The operation can be implemented for any hardware by providing the matching `BODY`.

> END