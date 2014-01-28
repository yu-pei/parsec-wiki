For PaRSEC-generated traces that fit comfortably within the limits of your system memory, there is a Python interface for converting and interacting with them. It's called Trace Tables, because every part of the trace is stored as part of a table data structure (i.e., a matrix) made possible by the open-source Python library pandas ([http://pandas.pydata.org/](http://pandas.pydata.org/)).

There are two fundamental components to the PaRSEC Trace Tables system - a converter library, **pbt2ptt**, written in Cython that interfaces directly with the C-level dbpreader library, and a Python-only library, **parsec_trace_tables.py**, designed for interfacing with the converted traces. It will commonly be necessary to use both of these libraries at the same time, first to convert a binary trace file to the Trace Tables format, and then to access the data.

The most basic way of performing this conversion is to run the converter library as a stand-alone program over the binary trace files that comprise an entire PaRSEC trace. For a shared-memory trace, there should be only one binary file per trace, but for a distributed trace there will be one binary file *per rank*. For each full trace, whether one file or many, you should invoke the pbt2ptt converter by passing the names of all of the files for one and only one trace as arguments on the command line.

For example:

SHARED-MEMORY (ONE BINARY FILE PER TRACE):


```
#!Bash

./pbt2ptt.py testing_dpotrf.prof-59mfcH
./pbt2ptt.py testing_dpotrf.prof-EjAi0I
./pbt2ptt.py testing_dpotrf.prof-L41nAM
```

DISTRIBUTED-MEMORY (ONE BINARY FILE PER RANK):


```
#!Bash

./pbt2ptt testing_dpotrf.prof-S7HmvJ testing_dpotrf.prof-PGpOS0 testing_dpotrf.prof-yr8jcS testing_dpotrf.prof-wiRjgs
```

**Note: It is important that you carefully provide the binary files of all ranks from a distributed trace if you wish to use the basic converter. If you leave out any of the ranks, or add ranks from a separate trace, the trace conversion may fail, or may even succeed without complaint, leaving you with a broken/corrupted set of trace tables, or no trace tables at all.**