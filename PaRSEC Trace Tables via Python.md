[TOC]

# **Intro** #

For PaRSEC Binary Trace (PBT) files that fit comfortably within the limits of your system memory, there is a Python interface for converting and interacting with them. It's called PaRSEC Trace Tables (PTT), because all parts of the trace are stored as part of a tabular data structure (i.e., a matrix). This extremely powerful and flexible approach is made possible by the open-source Python library pandas ([http://pandas.pydata.org/](http://pandas.pydata.org/)).

There are two fundamental components to the Trace Tables system - a converter library, **pbt2ptt**, written in Cython that interfaces directly with the C-level dbpreader library, and a Python-only library, **parsec_trace_tables.py**, designed for interfacing with the converted traces. It will commonly be necessary to use both of these libraries at the same time, first to convert a binary trace file to the Trace Tables format, and then to access the data.


**Note:** all command line instructions on this page assume that you have the Python scripts in your PATH environment variable. You may *source* the Bash script found in your PaRSEC build directory at **tools/profiling/python/utilities/bash.env** to automatically set up your environment.

# **Rudimentary Conversion and Load** #

## Basic Python Interface Conversion ##

The most basic way of performing this conversion is to import the converter library in a Python script and call its methods on the binary trace files that comprise an entire PaRSEC trace. For a shared-memory trace, there will be only one binary file per trace, but for a distributed trace there will be one binary file *per rank*. Whether one file or many, you should invoke the pbt2ptt converter by passing a list of *all* of the filenames for **one and only one** trace.

SHARED-MEMORY (ONE BINARY FILE PER TRACE):
```
#!Python
import pbt2ptt

pbt2ptt.convert(["testing_dpotrf.prof-59mfcH"])
pbt2ptt.convert(["testing_dpotrf.prof-9C19zV"])
pbt2ptt.convert(["testing_dpotrf.prof-4xACDt"])
```

DISTRIBUTED-MEMORY (ONE BINARY FILE PER RANK):
```
#!Python
import pbt2ptt

# 4 ranks
pbt2ptt.convert(["testing_dpotrf.prof-S7HmvJ", "testing_dpotrf.prof-PGpOS0", 
                 "testing_dpotrf.prof-yr8jcS", "testing_dpotrf.prof-wiRjgs"])
# 2 ranks
pbt2ptt.convert(["testing_dpotrf.prof-kVDDBr", "testing_dpotrf.prof-L41nAM"])
```

**It is important that you carefully provide the binary files of *all* ranks of a distributed trace if you wish to use the basic converter. If you leave out any of the ranks, or add rank trace files from a different trace, the trace conversion may fail, or may even succeed without complaint, leaving you with a broken/corrupted set of trace tables, or no trace tables at all. 
Therefore, *it is recommended that you use instead the autoload sequence described later in this document.* **

## About the Converted Trace Files ##

The pbt2ptt converter will store your Trace Tables in a single file for the entire trace (whether shared or distributed memory), using the excellent HDF5 file format via the Python library PyTables [(http://www.pytables.org)](http://www.pytables.org). Use of the pbt2ptt Trace Tables converter requires that you have enough local storage to contain the new Trace Tables copy of the trace. 

If you do not provide an output file name for the converter, the converter will select the name of the first of the binary trace files provided at the command line, modify it slightly in order not to overwrite the original, and use that as the output filename. The output filenames will have a ".h5" extension in order to signify that they are a standard HDF5 file. It is recommended that you allow the converter to choose its own filename, or, better yet, use the supplied ptt_utils.py functionality that will rename your files to a more useful name.

HDF5 is a very common big-data storage format, and as such the trace file may easily be read by many different languages and libraries. Only the Python parsec_trace_tables.py module is supported by the PaRSEC team, however.


## Basic Python Load ##

To load an already-converted Trace Tables file into a Python script, the most basic method is to pass the filename to the from_hdf function in the parsec_trace_tables module:

```
#!Python
import parsec_trace_tables as ptt  # note that the 'ptt' shorthand is highly recommended

my_trace = ptt.from_hdf("testing_dpotrf-59mfcH.h5")
print(my_trace.sched)
```

#### Load from Binary Format ####

To read a trace directly from the binary file(s), you use the pbt2ptt module function "read()":

```
#!Python
import parsec_trace_tables as ptt
import pbt2ptt

# note that read() takes a list of filenames even if there is only one file
my_trace = pbt2ptt.read(["testing_dpotrf.prof-59mfcH"])
print(my_trace.sched)
```

Alternatively, if you wish to first convert the binary trace to a Trace Tables file on local storage, you may call from_hdf() with the return value of convert():

```
#!Python
import parsec_trace_tables as ptt
import pbt2ptt

# convert returns the new PTT filename after writing the new PTT file to local storage
ptt_filename = pbt2ptt.convert(["testing_dpotrf.prof-S7HmvJ", "testing_dpotrf.prof-PGpOS0", 
                                "testing_dpotrf.prof-yr8jcS", "testing_dpotrf.prof-wiRjgs"])
my_trace = pbt2ptt.from_hdf(ptt_filename)
print(my_trace.sched)
```

**HOWEVER... It would be better if you ignored all of the above options for conversion and loading, and skipped straight to the section below about ptt_utils.py!**

# **ptt_utils.py** - the *Preferred* Method for Conversion and Load #

While the above methods (both command line and Python scripting) for converting and loading the PaRSEC Trace Tables are fully supported, it is recommended that you do not use them under most circumstances. They require the user to manually curate his or her set of trace files: to keep distributed traces grouped properly; to name and organize trace files appropriately for later access; to add clumsy conditions into his/her scripts to access the unconverted binary files the first time and the converted files all subsequent times in order to avoid the delays associated with conversion. 

Instead, it is recommend that you use the ptt_utils.py utility to transparently convert and open your traces. This utility provided the following functionality and benefits:

1. Algorithmically groups binary trace files from different ranks of the same run, reducing user error.
2. Converts multiple traces simultaneously on multiple cores, reducing wait time.
3. Configurably renames trace files to provide basic trace information as part of the filename, reducing user confusion.
4. Compresses existing Trace Tables HDF5 files in parallel, to reduce wait time and disk space usage.
5. Transparently loads from converted traces, if they exist, when provided the binary trace filenames, reducing script complexity and clutter.

The fifth benefit is the first benefit that most users will notice, as the command line and Python interfaces are both dramatically simplified. The method for almost every command line-based operation is "run ptt_utils.py over all files," and the method for almost every Python scripting operation is "call autoload_traces() with a list of all filenames." 


## Conversion and Load in Python ##
For instance, you may load many trace files of mixed types (PBT and PTT) simply by providing the entire list of filenames to autoload_traces():

```
#!Python
import ptt_utils

my_traces = ptt_utils.autoload_traces(["testing_dpotrf-59mfcH.h5",   "testing_dpotrf.prof-S7HmvJ",
                                       "testing_dpotrf.prof-PGpOS0", "testing_dpotrf.prof-yr8jcS",
                                       "testing_dpotrf.prof-wiRjgs", "testing_dpotrf.prof-kVDDBr",
                                       "testing_dpotrf.prof-kVDDBr", "testing_dpotrf.prof-L41nAM",
                                       "testing_dpotrf.prof-4xACDt", "testing_dpotrf.prof-9C19zV"])
```

By default, autoload_traces converts all supplied PBTs to PTTs before loading. By default, it allows the binary trace (PBT) file to remain on disk after conversion. It does not rename the files to enhance their filenames by default, though this functionality can be enabled easily and is recommended.

## Conversion and Filename Enhance at the Command Line ##

You may also convert PBTs to PTTs by providing a mixed type list to the ptt_utils.py script at the command line. The command line interface **DOES enhance filenames by default**, unlike autoload_traces().

```
#!Bash
ptt_utils.py testing_dpotrf-59mfcH.h5 testing_dpotrf.prof-kVDDBr testing_dpotrf.prof-L41nAM testing_dpotrf.prof-4xACDt

```

To convert at the command line without the default filename enhancing, simply supply the appropriate flag:

```
#!Bash
ptt_utils.py --no-enhance testing_dpotrf-59mfcH.h5 testing_dpotrf.prof-kVDDBr testing_dpotrf.prof-L41nAM testing_dpotrf.prof-4xACDt
```

## PTT Compression at the Command Line ##
Similarly, it is possible to compress PTT files from the command line with ptt_utils.py:
```
#!Bash
ptt_utils.py --compress *.h5 other_dir/testing_dpotrf-59mfcH.h5 other_dir/testing_dpotrf-kVDDBr.h5
```

# Usage #