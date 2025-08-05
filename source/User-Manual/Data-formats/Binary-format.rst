.. _binaryformat:
.. role:: raw-html-m2r(raw)
   :format: html

Binary Format
========================

.. image:: ../../_static/images/recordingdata/binary/header.png
  :alt: Binary data file icons

.. csv-table:: This is the default format for the Open Ephys GUI. Continuous data is stored in flat binary files, and everything else is in JSON / numpy format. Takes advantage of widely used open standards to maximize compatibility with existing and future analysis tools.
   :widths: 18, 80

   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Aarón Cuevas López, Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/main/Source/Processors/RecordNode/BinaryFormat"


**Advantages**

* Continuous data is stored in a compact format of tiled 16-bit integers, which can be memory mapped for efficient loading.

* Additional files are stored as JSON or NumPy data format, which can be read using :code:`numpy.load` in Python, or the `npy-matlab <https://github.com/cortex-lab/npy_matlab>`__ package.

* The format has no limit on the number of channels that can be recorded simultaneously.

* Continuous data files are immediately compatible with most spike sorting packages.

**Limitations**

* Requires slightly more disk space because it stores one 64-bit sample number and one 64-bit timestamp for every sample.

* Continuous files are not self-contained, i.e., you need to know the number of channels and the "bit-volts" multiplier in order to read them properly.

* It is not robust to crashes, as the NumPy file headers need to be updated when recording is stopped.

File organization
####################

Within a Record Node directory, data for each **experiment** is contained in its own sub-directory. The experiment number is incremented whenever acquisition is stopped, which will reset the timestamps/sample numbers to zero. A new :code:`settings.xml` file will be created for every experiment.

Experiment directories are further sub-divided for individual **recordings**. The recording number is incremented whenever recording is stopped, which will not reset the timestamps/sample numbers. Data from multiple recordings within the same experiment will have internally consistent timestamps/sample numbers, and will use the same :code:`settings.xml` file.

.. image:: ../../_static/images/recordingdata/binary/organization.png
  :alt: Binary data directory structure
  :width: 300

A recording directory contains sub-directories for **continuous**, **events**, and **spikes** data. It also contains a :code:`structure.oebin`, which is a JSON file detailing channel information, channel metadata, and event metadata descriptions.

Format details
################

Continuous
----------------

Continuous data is written separately for each stream within a processor (a block of synchronously sampled channels):

.. image:: ../../_static/images/recordingdata/binary/continuous.png
  :alt: Binary data continuous format
  :width: 300

Each **continuous** directory contains the following files:

* :code:`continuous.dat`: A simple binary file containing *N* channels x *M* samples signed 16-bit integers in little-endian format. Data is saved as :code:`ch1_samp1, ch2_samp1, ... chN_samp1, ch1_samp2, ch2_samp2, ..., chN_sampM`. The value of the least significant bit needed to convert the signed 16-bit integers to physical units is specified in the :code:`bitVolts` field of the relevant channel in the :code:`structure.oebin` JSON file. For "headstage" channels, multiplying by :code:`bitVolts` converts the values to microvolts, whereas for "ADC" channels, :code:`bitVolts` converts the values to volts.

* :code:`sample_numbers.npy`: A numpy array containing *M* 64-bit integers that represent the index of each sample in the :code:`.dat` file since the start of acquisition. **Note:** This file was called :code:`timestamps.npy` in GUI version 0.5.X. To avoid ambiguity, "sample numbers" always refer to integer sample index values starting in version 0.6.0.

* :code:`timestamps.npy`: A numpy array containing *M* 64-bit floats representing the global timestamps in seconds relative to the start of the Record Node's main data stream (assuming this stream was synchronized before starting recording). **Note:** This file was called :code:`synchronized_timestamps.npy` in GUI version 0.5.X. To avoid ambiguity, "timestamps" always refer to float values (in units of seconds) starting in version 0.6.0.

Events
-------

Event data is organized by stream and by "event channel" (typically :code:`TTL`). Each event channel records the states of multiple TTL lines.

.. image:: ../../_static/images/recordingdata/binary/events.png
  :alt: Binary data events format
  :width: 300

Directories for TTL event channels include the following files:

* :code:`states.npy`:  numpy array of *N* 16-bit integers, indicating ON (+CH_number) and OFF (-CH_number) states **Note:** This file was called :code:`channel_states.npy` in GUI version 0.5.X. 

* :code:`sample_numbers.npy` Contains *N* 64-bit integers indicating the sample number of each event since the start of acquisition. **Note:** This file was called :code:`timestamps.npy` in GUI version 0.5.X. To avoid ambiguity, "sample numbers" always refer to integer sample index values starting in version 0.6.0.

* :code:`timestamps.npy` Contains *N* 64-bit floats indicating representing the global timestamp of each event in seconds relative to the start of the Record Node's main data stream (assuming this stream was synchronized before starting recording). **Note:** This file did not exist in GUI version 0.5.X. Synchronized (float) timestamps for events first became available in version 0.6.0.

* :code:`full_words.npy`: Contains *N* 64-bit integers containing the "TTL word" consisting of the current state of *all* lines when the event occurred

Text events
^^^^^^^^^^^^

Text events are routed through the GUI's Message Center, and are stored in a directory called :code:`MessageCenter`. They contain the following files:

* :code:`text.npy`: numpy array of *N* strings

* :code:`sample_numbers.npy` Contains *N* 64-bit integers indicating the sample number of each text event on the Record Node's main data stream. **Note:** This file was called :code:`timestamps.npy` in GUI version 0.5.X. To avoid ambiguity, "sample numbers" always refer to integer sample index values starting in version 0.6.0.

* :code:`timestamps.npy` Contains *N* 64-bit floats indicating representing the global timestamp of each text event in seconds relative to the start of the Record Node's main data stream. **Note:** This file did not exist in GUI version 0.5.X. Synchronized (float) timestamps for events first became available in version 0.6.0.


Spikes
--------

Spike data is organized first by stream and then by electrode.

.. image:: ../../_static/images/recordingdata/binary/spikes.png
  :alt: Binary data spikes format
  :width: 300

Each **electrode** directory contains the following files:

* :code:`waveforms.npy`: numpy array with dimensions *S* spikes x *N* channels x *M* samples containing the spike waveforms

* :code:`sample_numbers.npy`: numpy array of *S* 64-bit integers containing the sample number corresponding to the peak of each spike. **Note:** This file was called :code:`timestamps.npy` in GUI version 0.5.X. To avoid ambiguity, "sample numbers" always refer to integer sample index values starting in version 0.6.0.

* :code:`timestamps.npy`: numpy array of *S* 64-bit floats containing the global timestamp in seconds corresponding to the peak of each spike (assuming this stream was synchronized before starting recording). **Note:** This file did not exist in GUI version 0.5.X. Synchronized (float) timestamps for spikes first became available in version 0.6.0.

* :code:`clusters.npy`: numpy array of *S* unsigned 16-bit integers containing the sorted cluster ID for each spike (defaults to 0 if this is not available).

More detailed information about each electrode is stored in the :code:`structure.oebin` JSON file.


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

Each :code:`Recording` object provides access to the continuous data, events, spikes, and metadata for the associated recording. To read the continuous data samples for the first data stream, you can use the following code:
 
.. code-block:: python

   samples = recordings[0].continuous[0].get_samples(start_sample_index=0, end_sample_index=10000)
   print(f'Stream name: {recordings[0].continuous[0].metadata("stream_name")}')

This will automatically scale the data into microvolts before returning it.

To read the events for the same recording:

.. code-block:: python

   events = recordings[0].events

This will load the events for all data streams into a Pandas :code:`DataFrame` with the following columns:

* :code:`line` - the TTL line on which this event occurred
* :code:`sample_number` - the sample index at which this event occurred
* :code:`timestamp` - the global timestamp (in seconds) at which this event occurred (defaults to -1 if all streams were not synchronized)
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
      np.searchsorted(recordings[0].continuous[0].timestamps, 
                      event_timestamp)


For more information on how to use the :code:`open-ephys-python-tools` library, check out this `README <https://github.com/open-ephys/open-ephys-python-tools/tree/main/src/open_ephys/analysis>`__

Using :code:`SpikeInterface`
--------------------------------------

You can also load data from the Open Ephys Binary format via `SpikeInterface <https://spikeinterface.readthedocs.io/en/stable/>`__, using the :code:`read_openephys()` method.

Reading data in Matlab
#######################

Use the `open-ephys-matlab-tools <https://github.com/open-ephys/open-ephys-matlab-tools>`__ library, available via the `Matlab File Exchange <https://www.mathworks.com/matlabcentral/fileexchange/122372-open-ephys-matlab-tools>`__.