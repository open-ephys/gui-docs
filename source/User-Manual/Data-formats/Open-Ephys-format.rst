.. _openephysformat:
.. role:: raw-html-m2r(raw)
   :format: html


Open Ephys Format
========================

.. image:: ../../_static/images/recordingdata/open-ephys/header.png
  :alt: Open Ephys data file icons

.. csv-table:: This is the original format used by the Open Ephys GUI. It is designed with redundancy in mind, so that data can be readily recovered even if the GUI crashes during acquisition. However, because data for each electrode is stored in a separate file, it doesn't scale to high channel count recordings. All files are stored in a single directory, with the file names used to identify the data source.
   :widths: 18, 80

   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Josh Siegle, Aarón Cuevas López"
   "*Source Code*", "https://github.com/open-ephys-plugins/open-ephys-data-format"

**Advantages**

* Data is stored in blocks with well-defined record markers, meaning data recovery is possible even if files are truncated by a crash.

* The file header contains all the information required to load the file.

**Limitations**

* Windows imposes a limit on the number files that can be written to simultaneously, meaning the Open Ephys format is not compatible with very high-channel-count recordings (>8000 channels).

* In order to achieve robustness, files contain redundant information, meaning they use extra space and take longer to load.

File organization
####################

All data files are stored within the same Record Node directory, with a completely flat hierarchy. Files for different **experiments** have a number appended after an underscore (starting with :code:`_2` for the second experiment in a session). Data from different **recordings** is distinguished by the :code:`recording index` values within each :code:`.spikes`, :code:`continuous`, or :code:`.events` file.

.. image:: ../../_static/images/recordingdata/open-ephys/organization.png
  :alt: Open Ephys data directory structure
  :width: 300

Each Record Node directory also contains at least one :code:`structure.openephys` file, an XML file with metadata about the available files, as well as :code:`messages.events`, a text file containing text events saved by the GUI.

Format details
################

Headers
---------

All headers are 1024 bytes long, and are written as a MATLAB-compatible string with the following fields and values:

* format = 'Open Ephys Data Format'

* version = 0.4

* header_bytes = 1024

* description = '(String describing the header)'

* date_created = 'dd-mm-yyyy hhmmss'

* channel = '(String with channel name)'

* channelType = '(String describing the channel type)'

* sampleRate = (integer sampling rate)

* blockLength = 1024

* bufferSize = 1024

* bitVolts = (floating point value of microvolts/bit for headstage channels, or volts/bit for ADC channels)

For those not using MATLAB, each header entry is on a separate line with the following format:

* field names appear after a "header." prefix

* field names and values are separated by " = "

* the value of the field is in plain text, with intended strings enclosed in single quotes (as demonstrated above)

* each line is terminated with a semicolon

Continuous
----------------

Continuous data for each channel is stored in a separate :code:`.continuous` file, identified by the processor ID (e.g. :code:`100`), stream name, and channel name (e.g. :code:`CH1`). After the 1024-byte header, continuous data is organized into "records," each containing 1024 samples.

.. image:: ../../_static/images/recordingdata/open-ephys/continuous.png
  :alt: Open Ephys data continuous format
  :width: 300

Each record is 2070 bytes long, and is terminated by a 10-byte record marker (0 1 2 3 4 5 6 7 8 255).


Events
-------

Events for each stream are stored in a :code:`.events` file. Each "record" contains the data for an individual event stored according to the following scheme:

.. image:: ../../_static/images/recordingdata/open-ephys/events.png
  :alt: Open Ephys data events format
  :width: 300


Spikes
--------

Data from each electrode is saved in a separate file. The filename is derived from the electrode name with spaces removed (e.g., "Electrode1") and the data stream name.

Each record contains an individual spike event (saved for one or more channels), and is written in the following format:

.. image:: ../../_static/images/recordingdata/open-ephys/spikes.png
  :alt: Open Ephys data spikes format
  :width: 300

Since the samples are saved as 16-bit unsigned integers, converting them to microvolts involves subtracting 32768, dividing by the gain, and multiplying by 1000.

Reading data in Python
#######################


Using :code:`open-ephys-python-tools`
--------------------------------------

The recommended method for loading data in Python is via the `open-ephys-python-tools <https://github.com/open-ephys/open-ephys-python-tools>`__ package, which can be installed via :code:`pip`.

First, create a :code:`Session` object that points to the top-level data directory:

.. code-block:: python

   from open_ephys.analysis import Session

   session = Session(r'C:\Users\example_user\Open Ephys\2025-08-03_10-21-22')

The :code:`Session` object provides access to data inside each Record Node. If you only had one Record Node in your signal chain, you can find its recordings as follows:

.. code-block:: python

   recordings = session.recordnodes[0]

:code:`recordings` is a list of :code:`Recording` objects, which is a flattened version of the original Record Node directory. For example, if you have three "experiments," each with two "recordings," there will be six total :code:`Recording` objects in this list. The one at index 0 will be recording 1 from experiment 1, index 1 will be recording 2 from experiment 1, etc.

Each :code:`Recording` object provides access to the continuous data, events, spikes, and metadata for the associated recording. To read the continuous data samples for the first data stream, you need to first set the sample range (to prevent all samples from being loaded into memory), and then call :code:`get_samples()`:
 
.. code-block:: python
   
   recordings[0].set_sample_range([10000, 50000])
   samples = recordings[0].continuous[0].get_samples(start_sample_index=0, end_sample_index=10000)
   print(f'Stream name: {recordings[0].continuous[0].metadata("stream_name")}')

This will automatically scale the data into microvolts before returning it.

To read the events for the same recording:

.. code-block:: python

   events = recordings[0].events

This will load the events for all data streams into a Pandas :code:`DataFrame` with the following columns:

* :code:`line` - the TTL line on which this event occurred
* :code:`sample_number` - the sample index at which this event occurred
* :code:`processor_id` - the ID of the processor from which this event originated
* :code:`stream_index` - the index of the stream from which this event originated
* :code:`stream_name` - the name of the stream from which this event originated
* :code:`state` - 1 or 0, to indicate whether this event is associated with a rising edge (1) or falling edge (0)

The fastest way to find the nearest continuous sample to a particular event is by using the :code:`np.searchsorted` function:

.. code-block:: python

   import numpy as np
   event_index = 100 # the index of the event you're interested in
   event_timestamp = events.iloc[event_index].timestamp 
   nearest_continuous_sample_index = \
      np.searchsorted(recordings[0].continuous[0].sample_numbers, 
                      event_sample_number)


For more information on how to use the :code:`open-ephys-python-tools` library, check out this `README <https://github.com/open-ephys/open-ephys-python-tools/tree/main/src/open_ephys/analysis>`__

Using :code:`SpikeInterface`
--------------------------------------

You can also load data from the Open Ephys Binary format via `SpikeInterface <https://spikeinterface.readthedocs.io/en/stable/>`__, using the :code:`read_openephys()` method.

Reading data in Matlab
#######################

Use the `open-ephys-matlab-tools <https://github.com/open-ephys/open-ephys-matlab-tools>`__ library, available via the `Matlab File Exchange <https://www.mathworks.com/matlabcentral/fileexchange/122372-open-ephys-matlab-tools>`__.