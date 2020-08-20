.. _binaryformat:
.. role:: raw-html-m2r(raw)
   :format: html

Binary format
========================

This format stores the continuous data in plain binary files, while all other data is stored in numpy arrays, easy to read in Python and which can be easily converted to plain binary arrays just by stripping their header.

Recording folders
#################

Each recording is stored in a subfolder called :code:`experiment<E>/recording<R>/`

Where :code:`E` denotes the experiment number and :code:`R` the recording number. This folder contains the following elements:

* :code:`continuous/`: Folder containing continuous data

* :code:`events/`: Folder containing event data

* :code:`spikes/`: Folder containing recorded spikes

* :code:`structure.oebin`: JSON file detailing channel information, channel metadata and event metadata descriptions. Contains a field for each of the recorded elements detailing their folder names, samplerate, channel count and other needed information.

* :code:`sync_messages.txt`: Text file with the synchronization messages that contain the start timestamp of the different processors

Data folders
#############

Continuous data
----------------

Since the GUI allows acquisition from different sources, which might have different sample rates, and recording can be done from the sources or any processor down the line, for each recorded processor the format creates a fileset per combination of recorded processor and data source. These filesets are stored in subfolders whose name can be:

* :code:`Processor_Name-<S>` for source processors

* :code:`Processor_Name-<P>_<S>` for other processors

Where the :code:`Processor_Name` is the one from the processor being recorded, :code:`S` represents a :code:`ProcessorID.SubprocessorIDX` of the source that originated the data and :code:`P` is the ID of the processor being recorded.

Each continuous filesets contains the following files:

* :code:`continuous.dat`: A simple binary file containning Nchans x Nsamples 16 bit integers in little endian format. 

* :code:`timestamps.npy`: A numpy array containing Nsamples 64bit integers that represent the timestamps for each sample.

The continuous data is saved as :code:`ch1_samp1, ch2_samp1, ... chN_samp1, ch1_samp2, ch2_samp2, ..., chN_sampM`. The value of the least significant bit needed to convert the 16-bit integres to microvolts is specified in the :code:`bitVolts` field of the relevant channel in the :code:`structure.oebin` JSON file.

Spikes
--------

For each processor that records spikes, a subfolder is created of the format

:code:`Processor_Name-<P>_<S>`

Where, similarly to the continuous case, :code:`P` denotes the ID of the processor which is recording the spikes, while :code:`S` denotes the ID of the node that is actually extracting the spike data from the continuous signal.

For convenience, spikes with the same configuration (same source, same number of channels and same metadata) are grouped into single datasets in folders called :code:`spike_group_N`. Each spike group contains the following files:

* :code:`spike_waveforms.npy`: numpy array of :code:`Nspikes` x :code:`Nchannels` x :code:`Nsamples` int16s containing the spike waveforms

* :code:`spike_times.npy`: numpy array of :code:`Nspikes` int64s containing the timestamps corresponding to the peak of each spike

* :code:`spike_electrode_indices.npy`: numpy array of :code:`Nspikes` unit16 specifying, for each spike in the group, which of the electrodes grouped in it originated each spike

* :code:`spike_clusters.npy`: numpy array of :code:`Nspikes` unit16 containing the sorted cluster id for which spike pertains, if the spike source processor performed spike sorted, all 0 otherwise.

* :code:`metadata.npy`: Optional. If the spikes contain metadata fields, they will be stored as an array of :code:`Nspikes` lists of fields or, if there is just one field, a Nspikes x Length_of_field array of the relevant type.

Detailed information about the electrodes contained in each spike group as well as the metadata fields, if any, is stored in the :code:`structure.oebin` JSON file.

Events
-------

Similarly to continuous data and spikes, each processor that generates events creates its own subfolder names

:code:`Processor_Name-<S>`

With :code:`S` being the :code:`ProcessorId.SubprocessorIDX` of the source processor. Each event channel the processor produces will have its own folder named :code:`TEXT_group_<N>`, :code:`BINARY_group_<N>` or :code:`TTL_<N>`, depending on the type of event.

Common files in all types of events are:

* :code:`timestamps.npy` Containing Nevents int64 for the timestamps of each event

* :code:`channels.npy` Containing Nevents uint16 indicating the virtual channel associated to each event (bit of the TTL word in their case)

* :code:`metadata.npy` Optional. If the events contain metadata fields, they will be stored as an array Nevents lists of fields or, if there is just one field, a Nspikes x Length_of_field array of the relevant type.

Text events
^^^^^^^^^^^^

* :code:`text.npy`: numpy array of :code:`Nevents` strings

Binary events
^^^^^^^^^^^^^^

* :code:`data_array.npy`: numpy array of :code:`Nevents` x :code:`data_length` elements of the relevant type

TTL events
^^^^^^^^^^

* :code:`channel_states.npy`:  Numpy array of Nevents int16. Each event will be written as +CH_number for rising events and -CH_number for falling events


* :code:`full_words.npy`: Numpy array of Nevents x log2(numBits) uint8 containing the binary representations of the full words received by the TTL source, in case they need to be treated as full qualified binary data.

