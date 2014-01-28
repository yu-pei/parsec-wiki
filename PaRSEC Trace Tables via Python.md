# **Intro** #

For PaRSEC-generated traces that fit comfortably within the limits of your system memory, there is a Python interface for converting and interacting with them. It's called Trace Tables, because all parts of the trace are stored as part of a tabular data structure (i.e., a matrix). This extremely powerful and flexible approach is made possible by the open-source Python library pandas ([http://pandas.pydata.org/](http://pandas.pydata.org/)).

There are two fundamental components to the PaRSEC Trace Tables system - a converter library, **pbt2ptt**, written in Cython that interfaces directly with the C-level dbpreader library, and a Python-only library, **parsec_trace_tables.py**, designed for interfacing with the converted traces. It will commonly be necessary to use both of these libraries at the same time, first to convert a binary trace file to the Trace Tables format, and then to access the data.

# **Conversion** #

## Basic Conversion ##

The most basic way of performing this conversion is to run the converter library as a stand-alone program over the binary trace files that comprise an entire PaRSEC trace. For a shared-memory trace, there should be only one binary file per trace, but for a distributed trace there will be one binary file *per rank*. For each full trace, whether one file or many, you should invoke the pbt2ptt converter by passing as command line arguments the names of *all* of the files for **one and only one** trace.

For example:

SHARED-MEMORY (ONE BINARY FILE PER TRACE):

```
#!Bash

./pbt2ptt.py testing_dpotrf.prof-59mfcH
./pbt2ptt.py testing_dpotrf.prof-EjAi0I
./pbt2ptt.py testing_dpotrf.prof-L41nAM
```

DISTRIBUTED-MEMORY (ONE BINARY FILE PER RANK, 4 RANKS):

```
#!Bash

./pbt2ptt testing_dpotrf.prof-S7HmvJ testing_dpotrf.prof-PGpOS0 testing_dpotrf.prof-yr8jcS testing_dpotrf.prof-wiRjgs
```

**Note: It is important that you carefully provide the binary files of all ranks from a distributed trace if you wish to use the basic converter. If you leave out any of the ranks, or add ranks from a separate trace, the trace conversion may fail, or may even succeed without complaint, leaving you with a broken/corrupted set of trace tables, or no trace tables at all.**

## Converted Trace Files ##

The pbt2ptt converter will store your Trace Tables in a single file for the entire trace (whether shared or distributed memory), using the excellent HDF5 file format via the Python library PyTables [(http://www.pytables.org)](http://www.pytables.org). Use of the pbt2ptt Trace Tables converter requires that you have enough local storage to contain the new Trace Tables copy of the trace. 

There is currently no option to provide an output file name for the converter. The converter will, by default, select the name of the first of the binary trace files provided at the command line, modify it slightly in order not to overwrite the original, and use that as the output filename. The output filenames will have a ".h5" extension in order to signify that they are a standard HDF5 file.

HDF5 is a very common big-data storage format, and as such the trace file may easily be read by many different languages and libraries. Only the Python parsec_trace_tables.py module is supported by the PaRSEC team, however.

## Basic Trace Tables Read/Open ##

To open a Trace Tables file from a Python script, the most basic method is to pass the filename to the from_hdf function in the parsec_trace_tables module:

```
#!Python
import parsec_trace_tables as ptt  # note that the 'ptt' shorthand is highly recommended

my_trace = ptt.from_hdf("testing_dpotrf-59mfcH.h5")
print(my_trace.sched)
```