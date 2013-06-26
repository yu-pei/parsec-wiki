The Python Profile reader, also named py_dbpreader, is a Python utility that uses the basic capabilities of the dbpreader tool written in C to expose the profiling information to Python code. 

To build the Python tool successfully, an installation of Python 2.7 or greater is recommended, and Cython 0.16 or greater is required. After building PaRSEC with the -fPIC flag set for all C compilers, enter the tools/profiling/python subdirectory of the source code, and run "python setup.py build_ext --inplace" to build py_dbpreader. The setup.py file may require some editing to get the build working on your machine. Contact pgaultne@utk.edu for help.

Once built, add the same tools/profiling/python directory to your PATH, PYTHONPATH, and LD_LIBRARY_PATH environment variables in order to ensure that the tool will run without issue.

For an example of how to read a profiling file directly, look at print_single_profile_event_types_by_thread.py. It loads a profiling file and uses the data within to print some basic information about the event types that were present during that run. 

For more advanced usage, you may want to use the available "TrialSet" Python class and its accompanying helper scripts, generate_trials.py and run_trials.py, which automate some of the common testing procedures used with PaRSEC. Further documentation on these is under development.
