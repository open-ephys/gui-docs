.. _binaryformat:
.. role:: raw-html-m2r(raw)
   :format: html

Binary format
========================

.. image:: ../../_static/images/recordingdata/binary/header.png
  :alt: Binary data file icons

.. csv-table:: This is the default format for the Open Ephys GUI. Continuous data is stored in flat binary files, and everything else is in JSON / numpy format. Takes advantage of widely used open standards to maximize compatibility with existing and future analysis tools. 
   :widths: 18, 80

   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "Yes"
   "*Key Developers*", "Aarón Cuevas López, Josh Siegle"
   "*Source Code*", "https://github.com/open-ephys/plugin-GUI/tree/master/Source/Processors/RecordNode/BinaryFormat"


**Advantages**

* Continuous data is stored in a compact format of tiled 16-bit integers, which can be memory mapped for efficient loading.

* Additional files are stored as JSON or NumPy data format, which can be read using :code:`numpy.load` in Python, or the `npy-matlab <https://github.com/kwikteam/npy-matlab>`__ package.

* The format is compatible with any combination of processors or subprocessors supported by the GUI

**Limitations**

* Requires slightly more disk space because it stores a 64-bit timestamp for every sample.

* Continuous files are not self-contained, i.e., you need to know the number of channels and the "bit-volts" multiplier in order to read them properly.

* It is not robust to crashes, as the NumPy file headers need to be updated when recording is stopped.

File organization
####################

Within a Record Node directory, data for each **experiments** (stop/start acquisition) is contained in its own sub-directory. Experiment directories are further sub-divided for individual **recordings** (stop/start recording).

.. image:: ../../_static/images/recordingdata/binary/organization.png
  :alt: Binary data directory structure
  :width: 300

A recording directory contains sub-directories for **continuous**, **events**, and **spikes** data. It also contains a :code:`structure.oebin`, which is a JSON file detailing channel information, channel metadata, and event metadata descriptions.

Format details
################

Continuous
----------------

Continuous data is grouped by sub-processor (a block of synchronously sampled channels):

.. image:: ../../_static/images/recordingdata/binary/continuous.png
  :alt: Binary data continuous format
  :width: 300

Each **continuous** directory contains the following files:

* :code:`continuous.dat`: A simple binary file containing *N* channels x *M* samples 16-bit integers in little-endian format. Data is saved as :code:`ch1_samp1, ch2_samp1, ... chN_samp1, ch1_samp2, ch2_samp2, ..., chN_sampM`. The value of the least significant bit needed to convert the 16-bit integers to physical units is specified in the :code:`bitVolts` field of the relevant channel in the :code:`structure.oebin` JSON file. For "headstage" channels, multiplying by :code:`bitVolts` converts the values to microvolts, whereas for "ADC" channels, :code:`bitVolts` converts the values to volts.

* :code:`timestamps.npy`: A numpy array containing *M* 64-bit integers that represent the timestamps for each sample.

* :code:`synchronized_timestamps.npy`: A numpy array containing *M* 64-bit floats representing the timestamps in seconds relative to the start of the master sub-processor.

Events
-------

Event data is organized by processor and by "event group" (e.g., :code:`TTL_<N>`). Each event group can contain data for multiple event channels.

.. image:: ../../_static/images/recordingdata/binary/events.png
  :alt: Binary data events format
  :width: 300

All types of events include the following files:

* :code:`timestamps.npy` Contains *N* 64-bit timestamps for each event

* :code:`channels.npy` Contains *N* unsigned 16-bit integers indicating the virtual channel associated to each event.

* :code:`metadata.npy` (optional) If the events contain metadata fields, they will be stored as an array of *N* lists of fields or, if there is just one field, a *N* x :code:`length_of_field` array of the relevant type.

Text events
^^^^^^^^^^^^

* :code:`text.npy`: numpy array of *N* strings

Binary events
^^^^^^^^^^^^^^

* :code:`data_array.npy`: numpy array of *N* x :code:`data_length` elements of the relevant type

TTL events
^^^^^^^^^^

* :code:`channel_states.npy`:  numpy array of *N* 16-bit integers, indicating ON (+CH_number) and OFF (-CH_number) states.

* :code:`full_words.npy`: numpy array of *N* x log2(numBits) unsigned 8-bit integers containing the binary representations of the full words received by the TTL source, in case they need to be treated as full qualified binary data.

Spikes
--------

Spike data is organized by processor and by "spike group" (a group of spike sources with the same number of channels). If, for example, you have stereotrodes and tetrodes within the same Spike Sorter plugin, the stereotrodes and tetrodes will appear in separate spike groups.

.. image:: ../../_static/images/recordingdata/binary/spikes.png
  :alt: Binary data spikes format
  :width: 300

Each **spike group** directory contains the following files:

* :code:`spike_waveforms.npy`: numpy array with dimensions *S* spikes x *N* channels x *M* samples containing the spike waveforms

* :code:`spike_times.npy`: numpy array of *S* 64-bit integers containing the timestamps corresponding to the peak of each spike

* :code:`spike_electrode_indices.npy`: numpy array of *S* unsigned 16-bit integers specifying which of the electrodes within the group the spike originated from

* :code:`spike_clusters.npy`: numpy array of *S* unsigned 16-bit integers containing the sorted cluster ID for each spike (defaults to 0 if this is not available).

* :code:`metadata.npy`: (optional) If the spikes contain metadata fields, they will be stored as an array of *S* lists of fields or, if there is just one field, a *S* x :code:`length_of_field` array of the relevant type.

Detailed information about the electrodes contained in each spike group as well as the metadata fields, if any, is stored in the :code:`structure.oebin` JSON file.


Reading data in Python
#######################

* **(recommended)** Create a :code:`Session` object using the `open-ephys-python-tools <https://github.com/open-ephys/open-ephys-python-tools>`__ package. The data format will be automatically detected.

* Create a :code:`File` object using the `pyopenephys <https://github.com/CINPLA/pyopenephys>`__ package.

* Use the :code:`DatLoad()` method from :code:`Binary.py` in the `open-ephys/analysis-tools <https://github.com/open-ephys/analysis-tools/blob/master/Python3/Binary.py>`__ repository.


Reading data in Matlab
#######################

* Use :code:`load_open_ephys_binary.m` from the `open-ephys/analysis-tools <https://github.com/open-ephys/analysis-tools/blob/master/load_open_ephys_binary.m>`__ repository.