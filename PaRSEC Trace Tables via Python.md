[TOC]

# **Intro** #

For PaRSEC Binary Trace (PBT) files that fit comfortably within the limits of your system memory, there is a Python interface for converting and interacting with them. It's called PaRSEC Trace Tables (PTT), because all parts of the trace are stored as part of a tabular data structure (i.e., a matrix). This extremely powerful and flexible approach is made possible by the open-source Python library pandas ([http://pandas.pydata.org/](http://pandas.pydata.org/)), which is therefore required to be present in your Python installation in order to use the Trace Tables.

There are two fundamental components to the Trace Tables system - a converter library, **pbt2ptt**, written in Cython that interfaces directly with the C-level dbpreader library, and a Python-only library, **parsec_trace_tables.py**, designed for interfacing with the converted traces. It will commonly be necessary to use both of these libraries, first to convert a binary trace file to the Trace Tables format, and then to access the data.

## software requirements as of February 2014 ##

PaRSEC Trace Tables is based on pandas, and requires the use of a number of other common Python libraries to compile and function.

* Cython 0.18+ is required for the compilation of the pbt2ptt library, which reads and converts the binary trace files.
* pandas 0.13+ is required for Python-side interface. See http://pandas.pydata.org/pandas-docs/stable/install.html for specific dependencies of pandas, including NumPy.
* PyTables 2.4+ is required for the converted HDF5 trace files.

* Matplotlib 1.3+ is necessary for many of the example scripts, which use it to plot the data, but is not required to use the basic PTT functionality in your own scripts.

Probably the easiest way to satisfy all of these dependencies at once is to install Continuum Analytics' "Anaconda" Python distribution (https://store.continuum.io/cshop/anaconda/), which is free and comes with all of the dependencies pre-installed.

On Mac OS, macports contains all the necessary software. 

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

### from Trace Tables (PTT) ###
To load an already-converted Trace Tables file into a Python script, the most basic method is to pass the filename to the from_hdf function in the parsec_trace_tables module:

```
#!Python
import parsec_trace_tables as ptt  # note that the 'ptt' shorthand is highly recommended

my_trace = ptt.from_hdf("testing_dpotrf-59mfcH.h5")
print(my_trace.sched)
```

### from Binary Format (PBT) ###

To read a trace directly from the binary file(s) without first converting in order to save the local storage space used by a converted PTT file, you use the pbt2ptt module function "read()":

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

**HOWEVER...** It will be better if you **ignore all of the above options for conversion and loading**, and skip straight to the section below about ptt_utils.py!

# **ptt_utils.py** - the *Preferred* Method for Conversion and Load #

While the above methods (both command line and Python scripting) for converting and loading the PaRSEC Trace Tables are fully supported, it is recommended that you do not use them under most circumstances. They require the user to manually curate his or her set of trace files: to keep distributed traces grouped properly; to name and organize trace files appropriately for later access; to add clumsy conditions into his/her scripts to access the unconverted binary files the first time and the converted files all subsequent times in order to avoid the delays associated with conversion. 

Instead, it is recommend that you use the ptt_utils.py utility to transparently convert and open your traces. This utility provided the following functionality and benefits:

1. Algorithmically groups binary trace files from different ranks of the same run, reducing user error.
2. Converts multiple traces simultaneously on multiple cores, reducing wait time.
3. Configurably renames trace files to provide basic trace information as part of the filename, reducing user confusion.
4. Compresses existing Trace Tables HDF5 files in parallel, to reduce disk space usage.
5. Transparently loads from converted traces, if they exist in the same directory, when provided the binary trace filenames, reducing script complexity and clutter.

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

# **Usage** #

## PTT **is** multiple pandas :) ###

The PaRSEC Trace Tables Python class is a relatively simple container for multiple [pandas](http://pandas.pydata.org) DataFrames (or tables) and multiple pandas Series (or dictionaries). It is highly recommended that you refer to the [pandas documentation](http://pandas.pydata.org/pandas-docs/stable/) for specifics on how to interact with the data. 

As a basic introduction, however, you can think of the pandas DataFrame as a 2-D matrix, with the columns specifying the different data types and their names/labels, and the rows being the data themselves - one row per set of related data elements (e.g., a single trace event, or the information about a single thread). A Series is basically a dictionary or hash table, though you can also think of it as being a one-row DataFrame. 

In addition to the pandas docs, you may find the demo_ptt.py script in the "tools/profiling/python/examples" directory to be a helpful and quick reference for some of the basic ways to interact with the data.

## PaRSEC Trace Tables Structure ##

The current table and dictionary types can be easily referenced from the **parsec_trace_tables.py** Python module file, but the table/DataFrame names as of this writing are:

* events
* nodes
* streams
* errors

and the dictionary/Series names are:

* event_types
* event_names
* information
* event_attributes

## Events ##

The events table is the most important, as it contains one row for every event in the trace. Not all events have all the same elements (i.e., not all rows have a non-null datum for every column), but all PaRSEC trace events currently have at least the following data elements:

* id
* node_id
* thread_id
* handle_id
* type
* begin
* end
* duration
* flags

Depending on the event, there may be additional data elements (columns), such as "kernel_type".

These event data can be accessed, like any other data element belonging to a given event, with the following common Python syntax:

```
#!Python
import ptt_utils

my_traces = ptt_utils.autoload_traces(['testing_dpotrf.prof-fjf93f'])
my_trace = my_traces[0]

my_trace.events.node_id # this will provide a Series of all node_ids for all events
my_trace.events.node_id.iloc[4] # this will provide the node_id for the event in row 4 (0 indexed)

# this next statement provides node_ids where the node_id == 4
my_trace.events.node_id[:][trace.events.node_id == 4] # this is a special pandas syntax for filtering data from a DataFrame

```

## Event Names & Types ##

There are two dictionary lookups to assist in filtering the PTT event data in a human-readable way. Every PaRSEC trace event has a "type," and these types are represented in the event data itself by integers. In order to perform a lookup by name, use the event_types dictionary to retrieve the integer type associated with a name. For a reverse lookup of a name by integer type, use the event_names dictionary:


```
#!Python
# we already have loaded a trace...

GEMM_type_int = my_trace.event_types['GEMM']
# now filter the events to view only the GEMM events
gemm_events = my_trace.events[:][my_trace.events.type == GEMM_type_int]

# now we lookup the type name of a randomly-selected event
random_event = my_trace.events.iloc[3435]
event_type_name = my_trace.event_names[random_event.type]

```

## describe() and dtypes ##

For further information about each part of the PaRSEC Trace Tables object, you can use both of the following pieces of code for each part:


```
#!Python

print(my_trace.events.dtypes)
print(my_trace.nodes.dtypes)
print(my_trace.information.dtypes)
# etc

# -- and --

print(my_trace.events.describe())
print(my_trace.threads.describe())
print(my_trace.event_types.describe())
# etc

```
