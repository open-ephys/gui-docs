.. _nwbdataformat:
.. role:: raw-html-m2r(raw)
   :format: html

NWB format
=====================


.. image:: ../../_static/images/recordingdata/nwb/header.png
  :alt: NWB data file icon

|

.. csv-table:: This is a data format based on the `NWB 1.0 specification <https://alleninstitute.github.io/nwb-api/index.html>`__, formerly maintained by the Allen Institute, but now deprecated. It will soon be replaced by the NWB 2.0 format. Since it is not available by default, it must be downloaded via the GUI's Plugin Installer.
   :widths: 18, 80

   "*Platforms*", "Windows, Linux, macOS"
   "*Built in?*", "No"
   "*Key Developers*", "Aarón Cuevas López"
   "*Source Code*", "https://github.com/open-ephys-plugins/NWBFormat"

.. caution:: The NWB 1.0 format has been deprecated. It will be replaced by NWB 2.0 in a future version of the GUI.

**Advantages**

* Data is stored in a single HDF5 file, with self-documenting internal structure.

* Can be read using standard HDF5 tools, such as `HDFView <https://www.hdfgroup.org/downloads/hdfview/>`__.

**Limitations**

* HDF5 files must be closed gracefully, so data may be irrecoverable if the GUI crashes during acquisition.

* The NWB 1.0 specification has been deprecated, and will not be developed further.

* The HDF5 C++ library is not thread-safe, so you cannot write to the NWB format from multiple Record Nodes simultaneously.

File organization
####################

Within a Record Node directory, data for each **experiment** (stop/start acquisition) is contained in a separate NWB file. Individual **recordings** are stored in separate groups inside the file.

|

.. image:: ../../_static/images/recordingdata/nwb/organization.png
  :alt: NWB data file structure
  :width: 300

|

Each NWB file also contains the following information:

* :code:`/file_create_date`: date + time in ISO format (text array)
* :code:`/identifier`: from text fields in recording options (text)
* :code:`/nwb_version`: 'NWB-1.0.6' (text)
* :code:`/session_description`: <blank string> (text)
* :code:`/session_start_time`: date + time in ISO format (text)

Format details
################

Continuous
----------------

Continuous data is grouped by sub-processor (a block of synchronously sampled channels):

|

.. image:: ../../_static/images/recordingdata/nwb/continuous.png
  :alt: NWB data continuous format
  :width: 300

|

Each **continuous** group is an NWB *ElectricalSeries* containing the following datasets:

* :code:`data`: *N* channels x *M* samples of 16-bit integers. The :code:`conversion` attribute stores the "bitVolts" value required to convert these values into volts.

* :code:`timestamps`: *M* 64-bit floats representing the timestamps (in seconds) for each sample.



Events
-------

Event data is organized by "event group" (e.g., :code:`TTL_<N>`). Each event group can contain data for multiple event channels.

|

.. image:: ../../_static/images/recordingdata/nwb/events.png
  :alt: NWB data events format
  :width: 300

|

TTL events include the following datasets:

* :code:`timestamps`: *N* 64-bit float representing the timestamps (in seconds) for each event

* :code:`data`: *N* event codes indicating ON (+CH_number) and OFF (-CH_number) states

* :code:`control`: *N* unsigned 8-bit integers representing virtual channel numbers


Spikes
--------

Spike data is organized by electrode.

|

.. image:: ../../_static/images/recordingdata/nwb/spikes.png
  :alt: NWB data spikes format
  :width: 300

|

Each **spikes** group contains the following datasets:

* :code:`data`: array with dimensions *S* spikes x *N* channels x *M* samples containing the spike waveforms. The :code:`conversion` attribute stores the "bitVolts" value required to convert these values into volts.

* :code:`timestamps`: *S* 64-bit floats containing the timestamps (in seconds) corresponding to the peak of each spike


Reading data in Python
#######################

* Create a :code:`Session` object using the `open-ephys-python-tools <https://github.com/open-ephys/open-ephys-python-tools>`__ package. The data format will be automatically detected.


Reading data in Matlab
#######################

For now, use Matlab's :code:`h5read` method to open datasets within an NWB file, e.g.:

.. code-block:: matlab

   data = h5read(filename, dataset)

NWB-specific loading functions are coming soon.



Full specification
#####################

For writing continuous data, we use the NWB :code:`ElectricalSeries`:

.. code-block:: 

	/acquisition/timeseries/recording<R>/continuous/processor<P>
	    ./ancestry: ['TimeSeries', 'ElectricalSeries'] (text array attr)
	    ./comments: <empty> (text attr)
	    ./description: <empty> (text attr)
	    ./neurodata_type: 'TimeSeries' (text attr)
	    ./source: Recorded processor name + <P> From data source processor name <P> (text attr)
	    ./help: 'Stores acquired voltage data from extracellular recordings' (text attr)
	    ./data: M samples x N channels (int16 array)
	        ./conversion: bitVolts (float32 attr)
	        ./resolution: 1/2^16 * bitVolts (float32 attr)
	        ./unit: 'volt' (text attr)
	    ./num_samples: M (int32, stored at end of recording)
	    ./timestamps: M timestamps relative to experiment start time (float64 array)
	        ./interval: 1 (int32 attr)
	        ./unit: 'seconds' (text attr)
	    ./oe_extra_info
	        ./schema_id: 'openephys:extra_info/' (text attr)
	        ./channel<C>
	            ./schema_id: 'openephys:<channel_info>/'
	            ./name: channel info object name (text attr)
	            ./description: channel info object description (text attr)
	            ./identifier: channel info object identifier (text attr)
	            ./source_index: index of this channel in source processor (uint16 attr)
	            ./source_type_index: index of this type of channel in source processor (uint16 attr)
	            ./channel_metadata
	                ./schema_id: 'openephys:<metadata>/' (text attr)
	                ./Field_<M>: L values or text string
	                    ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                    ./name: metadata field name (text attr)
	                    ./description: metadata description (text attr)
	                    ./identifier: metadata identifier (text attr)
 
For writing spike data, we use the NWB :code:`SpikeEventSeries`:

.. code-block:: 

	/acquisition/timeseries/recording<R>/spikes/electrode<E>
	    ./ancestry: ['TimeSeries', 'SpikeEventSeries'] (text array attr)
	    ./comments: <empty> (text attr)
	    ./description: electrode name (text attr)
	    ./neurodata_type: 'TimeSeries' (text attr)
	    ./source: Processor name + <P> (text attr)
	    ./help: 'Snapshorts of spike events from data' (attr, text)
	    ./data: X spikes x Y channels x Z samples (int16 array)
	        ./conversion: bitVolts (float32 attr)
	        ./resolution: 1/2^16 * bitVolts (float32 attr)
	        ./unit: 'volt' (text attr)
	    ./timestamps: X timestamps relative to experiment start time (float64 array)
	        ./interval: 1 (int32 attr)
	        ./unit: 'seconds' (text attr)
	    ./oe_extra_info
	        ./schema_id: 'openephys:oe_extra_info/'
	        ./name: channel info object name (text attr)
	        ./description: channel info object description (text attr)
	        ./identifier: channel info object identifier (text attr)
	        ./source_index: index of this channel in source processor (uint16 attr)
	        ./source_type_index: index of this type of channel in source processor (uint16 attr)
	        ./channel_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
	        ./spike_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
 
For writing messages, we use the NWB :code:`AnnotationSeries`:

.. code-block:: 

	/acquisition/timeseries/recording<R>/events/text<T>
	    ./ancestry: ['TimeSeries', 'AnnotationSeries'] (text array attr)
	    ./comments: <empty> (text attr)
	    ./description: channel info object description (text attr)
	    ./neurodata_type: 'TimeSeries' (text attr)
	    ./source: Processor name + <P> (text attr)
	    ./help: 'Time-stamped annotations about an experiment' (attr, text)
	    ./data: N messages as text array
	        ./conversion: 'NaN' (float32 attr)
	        ./resolution: 'NaN' (float32 attr)
	        ./unit: 'n/a' (text attr)
	    ./control: N uint8 representing virtual channel numbers. usually all zeros.
	    ./num_samples: N (int32, stored at end of recording)
	    ./timestamps: N timestamps relative to experiment start time (float64 array)
	        ./interval: 1 (int32 attr)
	        ./unit: 'seconds' (text attr)
	    ./oe_extra_info
	        ./schema_id: 'openephys:oe_extra_info/'
	        ./name: channel info object name (text attr)
	        ./description: channel info object description. Same as base class description (text attr)
	        ./identifier: channel info object identifier (text attr)
	        ./source_index: index of this channel in source processor (uint16 attr)
	        ./source_type_index: index of this type of channel in source processor (uint16 attr)
	        ./channel_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
	        ./spike_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
	 
For writing TTL events, we use a custom derived version of the NWB :code:`IntervalSeries` called :code:`TTLSeries`:

.. code-block:: 

	/acquisition/timeseries/recording<R>/events/ttl<T>
	    ./ancestry: ['TimeSeries', 'IntervalSeries', 'TTLSeries'] (text array attr)
	    ./comments: <empty> (text attr)
	    ./description: channel info object description (text attr)
	    ./neurodata_type: 'TimeSeries' (text attr)
	    ./source: Processor name + <P> (text attr)
	    ./help: 'Stores the start and stop times for TTL events' (attr, text)
	    ./data: N events [+(channel#) for event 'on', -(channel#) for event 'off'] (int8 array)
	        ./conversion: 'NaN' (float32 attr)
	        ./resolution: 'NaN' (float32 attr)
	        ./unit: 'n/a' (text attr)
	    ./control: N uint8 representing virtual channel numbers. Same as absolute value of data
	    ./num_samples: N (int32, stored at end of recording)
	    ./timestamps: N timestamps relative to experiment start time (float64 array)
	        ./interval: 1 (int32 attr)
	        ./unit: 'seconds' (text attr)
	    ./full_word: N x M uint8, where M is the number of bytes needed to fit the whole ttl word.
	        ./schema_id: 'openephys:full_word' (text attr)
	    ./oe_extra_info
	        ./schema_id: 'openephys:oe_extra_info/'
	        ./name: channel info object name (text attr)
	        ./description: channel info object description. Same as base class description (text attr)
	        ./identifier: channel info object identifier (text attr)
	        ./source_index: index of this channel in source processor (uint16 attr)
	        ./source_type_index: index of this type of channel in source processor (uint16 attr)
	        ./channel_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
	        ./event_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)

For writing Binary events, we use a custom derived version of the NWB :code:`Timeseries` called :code:`BinarySeries`:

.. code-block:: 

	/acquisition/timeseries/recording<R>/events/binary<T>
	    ./ancestry: ['TimeSeries', 'BinarySeries'] (text array attr)
	    ./comments: <empty> (text attr)
	    ./description: channel info object description (text attr)
	    ./neurodata_type: 'TimeSeries' (text attr)
	    ./source: Processor name + <P> (text attr)
	    ./help: 'Stores arbitrary binary data' (attr, text)
	    ./data: N events x M length of data, any kind of numeric type.
	        ./conversion: 'NaN' (float32 attr)
	        ./resolution: 'NaN' (float32 attr)
	        ./unit: 'n/a' (text attr)
	    ./control: N uint8 representing virtual channel numbers. Usually all zeros.
	    ./num_samples: N (int32, stored at end of recording)
	    ./timestamps: N timestamps relative to experiment start time (float64 array)
	        ./interval: 1 (int32 attr)
	        ./unit: 'seconds' (text attr)
	    ./oe_extra_info
	        ./schema_id: 'openephys:oe_extra_info/'
	        ./name: channel info object name (text attr)
	        ./description: channel info object description. Same as base class description (text attr)
	        ./identifier: channel info object identifier (text attr)
	        ./source_index: index of this channel in source processor (uint16 attr)
	        ./source_type_index: index of this type of channel in source processor (uint16 attr)
	        ./channel_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)
	        ./event_metadata
	            ./schema_id: 'openephys:<metadata>/' (text attr)
	            ./Field_<M>: L values or text string
	                ./schema_id: 'openephys:<text_metadata>' or 'openephys:<number_metadata>' (text attr) 
	                ./name: metadata field name (text attr)
	                ./description: metadata description (text attr)
	                ./identifier: metadata identifier (text attr)

