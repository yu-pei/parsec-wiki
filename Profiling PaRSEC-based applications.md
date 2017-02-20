In many instances it might be interesting to extract and use information about the task cost and performance. This allows analysis of the scheduling quality, the maintained memory locality and the time wasted due to scheduling decisions. Achieving this requires a complete recompilation of the PaRSEC source after enabling the PARSEC_PROF_TRACE CMake option.

In case you are on a system with accelerators and you need to be able to extract profiling information from the execution of tasks on these accelerators you have to hand edit the src/gpu_data.c file and add the information you want to profile to the ```parsec_cuda_trackable_events``` variable. Here is a quick description of the available values:

* PARSEC_PROFILE_CUDA_TRACK_EXEC show the task submission and retrieval time. This does not include the time required to move the data to and from the accelerator.

* PARSEC_PROFILE_CUDA_TRACK_DATA_IN track the movement of data from the main memory to the accelerator memory. This operation can be skipped if the data is already available on the accelerator memory due to a previous operation.

* PARSEC_PROFILE_CUDA_TRACK_DATA_OUT track the data movement from the accelerator memory back to the main memory after the completion of a task. This operation can be delayed until the data is needed by a task that will execute on the CPU.

* PARSEC_PROFILE_CUDA_TRACK_OWN show the threads that own the accelerator, in the sense that it will manage the tasks submission and data movements.

Once the desired options have been set, a complete rebuild of the software is necessary. You're now ready to use the profiling. Let's suppose you want to profile the ```dplasma/testing/testing_dpotrf``` test in a heterogeneous distributed environment. I'll take a small example so I will limit the number of processes to 2, the number of cores per process to 2 as well, the size of a tile to 200 and the size of the matrix to 4000. The command will look similar to the following:


```
#!shell

mpirun -np 2 dplasma/testing/testing_dpotrf -c 2 -t 200 -N 4000 -- --mca profile_filename demo

```

The result will look something like

```
#!shell

[****] TIME(s)      0.81082 : dpotrf	PxQ=   2 1   NB=  200 N=    4000 :      26.320778 gflops

```

Two files (one per process) are generated in the current directory (```demo-0.prof-<random number>```, and ```demo-1.prof-<random number>``` -- NB: if you used another name for the profile_filename argument above, that name, and not demo will be used. Similarly, the rank will go from 0 to NP-1). 

These files follow an internal binary format that contains realtime timestamps in the architecture of the local machine. As such, they are probably unsuitable to be transferred and analyzed at a remote place and must be converted. There are two alternatives to do so:

* directly convert from profile to PAJe [(external website)](http://paje.sourceforge.net/) / VITE [(external website)](http://vite.gforge.inria.fr/) traces using the dbp2paje converter in tools/profiling. This method is obsolete and is not recommended anymore
* convert the profile files to HDF5 files, and use python tools to analyze and convert this portable trace on other machines. This requires python and cython to work on the target machine, and is explained in more details below:

First, check that python is supported by your PaRSEC compilation from the output of CMake. It should not complain about python or cython not working. If it does, you need to upgrade your python / cython installation.

Second, load tools/profiling/python/utilities/bash.env (or fish.env if you use fish) in your shell environment to define the appropriate PYTHONPATH

```
#!shell
. tools/profiling/python/utilities/bash.env
```

Then, convert the profile files in the HDF5 file with the python script tools/profiling/python/profile2h5:

```
#!shell
> python2.7 tools/profiling/python/profile2h5 demo-0.prof-6NdC2R demo-1.prof-6NdC2R 
Processing ['demo-0.prof-6NdC2R', 'demo-1.prof-6NdC2R']
Generated: ./demo-hostname--4-3000-180-lfq-6NdC2R.h5
>
```

The generated file name features the hostname, some information about the run, and a random number. It is an HDF5 file that you can process with the tool of your choice. Pandas / NUMPY are excellent tools to handle this data.

Visual traces can be obtained with the tools/profiling/python/h5totrace.py: In its simplest call, one can simply issue:
```
#!shell
>  python2.7 tools/profiling/python/h5totrace.py --h5 demo-hostname-4-3000-180-lfq-6NdC2R.h5 --out demo.trace
Closing remaining open files:demo-hostname-4-3000-180-lfq-6NdC2R.h5...done
```

And the demo.trace file can be opened with any PAJe file format tool (e.g. VITE).

The h5totrace.py script allows also to
* filter events (remove events following a pattern or specific names)
* filter threads (avoid to display events that happened on threads that are not desired in the final output)
* extract human-readable names of tasks, using the DOT file (see how a DOT file can be generated here). This functionality allows to have better task names in the visual tools; without a DOT file to compute these names, a unique task id is provided, but connecting it to the original task name can be hard.
* summarize specific events (e.g. memory consumption related events) as counters.
* list event types names and simple statistics on the trace

See the help of the script to understand how these can be achieved.

The picture below is the trace corresponding to the above mentioned execution as showed using [Vite](http://vite.gforge.inria.fr/). Don't worry a normal trace will look much better, in this trace I kept the MPI event support, but removed the MPI events and threads. 

![Vite Trace](files/dpotrf.trace.png)