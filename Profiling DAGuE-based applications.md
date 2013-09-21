In many instances it might be interesting to extract and use information about the task cost and performance. This allows the analysis of the scheduling quality, the maintained memory locality and the time wasted due to scheduling decisions. Achieving this requires a complete recompilation of the DAGuE source after enabling the DAGUE_PROF_TRACE CMake option.

In case you are on a system with accelerators and you need to be able to extract profiling information from the execution of tasks on these accelerators you have to hand edit the src/gpu_data.c file and add the information you want to profile to the dague_cuda_trackable_events variable. Here is a quick description of the available values:

* DAGUE_PROFILE_CUDA_TRACK_EXEC show the task submission and retrieval time. This does not include the time required to move the data to and from the accelerator.

* DAGUE_PROFILE_CUDA_TRACK_DATA_IN track the movement of data from the main memory to the accelerator memory. This operation can be skipped if the data is already available on the accelerator memory due to a previous operation.

* DAGUE_PROFILE_CUDA_TRACK_DATA_OUT track the data movement from the accelerator memory back to the main memory after the completion of a task. This operation can be delayed until the data is needed by a task that will execute on the CPU.

* DAGUE_PROFILE_CUDA_TRACK_OWN show the threads that own the accelerator, in the sense that it will manage the tasks submission and data movements.

Once the desired options have been set, a complete rebuild of the software is necessary. You're now ready to use the profiling. Let's suppose you want to profile the dplasma/testing/testing_dpotrf test in a heterogeneous distributed environment. I'll take a small example so I will limit the number of processes to 2, the number of cores per process to 2 as well, the size of a tile to 200 and the size of the matrix to 4000. The command like will look similar to the following:


```
#!shell

mpirun -np 2 dplasma/testing/testing_dpotrf -c 2 -t 200 -N 4000

```

The result will look something like

```
#!shell

[****] TIME(s)      0.81082 : dpotrf	PxQ=   2 1   NB=  200 N=    4000 :      26.320778 gflops

```

Two files (one per process) are generated in the current directory (<app_name>.rank.profile). As you have compiled with support for the profiling, under tools/profiling you will find a conversion tool dbp2paje, which convert our internal profiling traces into a more standard format. Executing

```
#!shell
tools/profiling/dbp2paje testing_dpotrf.?.profile -o dpotrf
```

will write in the current directory the global trace file dpotrf.trace, file you can use with different visualizers to see the execution trace. The picture below is the trace corresponding to the above mentioned execution as showed using [Vite](http://vite.gforge.inria.fr/). Don't worry a normal trace will look much better, in this trace I kept the MPI event support, but removed the MPI events. If you want to play with Vite locally, I attached the trace used in this example [here](files/dpotrf.trace).

![Vite Trace](files/dpotrf.trace.png)
