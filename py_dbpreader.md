The Python Profile reader, also named py_dbpreader, is a Python utility that uses the basic capabilities of the dbpreader tool written in C to expose the profiling information to Python code. 

py_dbpreader must be built before it can be used. Unfortunately, the PaRSEC build process does not automatically build py_dbpreader. The py_dbpreader is commonly built in-place in the source directory, but it cannot be built without a libdague-base.a library to link with.

To build the Python tool successfully, an installation of Python 2.7 or greater is recommended, and Cython 0.16 or greater is required. NumPy and Matplotlib/PyPlot are recommended but not necessary. The easiest way to achieve this is to download and install [Enthought Canopy](https://www.enthought.com/store/) on the same machine that you are using to compile and run PaRSEC. It is relatively trivial to get an Academic License for Canopy Basic, but py_dbpreader currently has no dependencies on packages that cannot be enabled using Canopy Express.

Once the correct environment is available, build PaRSEC with the -fPIC flag set for all C compilers (in advanced mode, add "-fPIC" to CMAKE_CXX_FLAGS and CMAKE_C_FLAGS). Then, enter the tools/profiling/python subdirectory of the source code, and run "python setup.py build_ext --inplace" to build py_dbpreader. If your top-level build directory is not ../../../../build, append to the setup invocation an absolute or relative path to your top-level build directory, e.g. "python setup.py build_ext --inplace ../../../../pins_build". 
The setup.py script may require some editing to get the environment properly configured on your machine, depending on the locations of your BLAS and basic compiler libraries. Contact me at pgaultne@utk.edu for more help if needed.

A successful build will generate py_dbpreader.so in the current directory. Once built, add this tools/profiling/python directory to your PATH, PYTHONPATH, and LD_LIBRARY_PATH environment variables in order to ensure that the tool will run without issue.

For an example of how to read a profiling file directly, look at print_single_profile_event_types_by_thread.py. It loads a profiling file and uses the data within to print some basic information about the event types that were present during that run. 

For more advanced usage, you may want to use the available "TrialSet" Python class and its accompanying helper scripts, generate_trials.py and run_trials.py, which automate some of the common testing procedures used with PaRSEC. Further documentation on these is under development.
