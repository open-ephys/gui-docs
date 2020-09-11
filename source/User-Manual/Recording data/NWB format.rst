.. _nwbdataformat:
.. role:: raw-html-m2r(raw)
   :format: html

NWB format
=====================

From the NWB website:

.. epigraph::

	Neurodata Without Borders: Neurophysiology (NWB:N) is a project to develop a unified data format for cellular-based neurophysiology data, focused on the dynamics of groups of neurons measured under a large range of experimental conditions. The NWB:N team consists of neuroscientists and software developers who recognize that adoption of a unified data format is an important step toward breaking down the barriers to data sharing in neuroscience.

We are working on implementing the ability to write directly to an NWB file from the Open Ephys GUI. We're hoping that NWB becomes a common standard for neuroscience, so saving your data in NWB format will make it much easier to share with other researchers. It also comes with all the advantages (and disadvantages) of HDF5.

The official format specification can be found on GitHub as raw HTML files. We've adapted it for use with Open Ephys by extending some base classes of the NWB specification to hold extra data that might be provided by the software. Extension files for the official NWB API will be provided as per the standard. The resulting format is as follows:

Open Ephys NWB File Structure (based on version 1.0.6)
########################################################

**Top-level groups (always created):**

* /acquisition
* /analysis
* /epochs
* /general
* /processing
* /stimulus

**Top-level datasets (always created):**

* /file_create_date: date + time in ISO format (text array)
* /identifier: from text fields in recording options (text)
* /nwb_version: 'NWB-1.0.6' (text)
* /session_description: <blank string> (text)
* /session_start_time: date + time in ISO format (text)


/acquisition group:
All of the data generated during recording should be stored in this group.

For writing continuous data, we use the NWB `ElectricalSeries`:

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
 
For writing spike data, we use the NWB `SpikeEventSeries`:

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
 
For writing messages, we use the NWB `AnnotationSeries`:

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
	 
For writing TTL events, we use a custom derived version of the NWB `IntervalSeries` called `TTLSeries`:

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

For writing Binary events, we use a custom derived version of the NWB  `Timeseries` called `BinarySeries`:

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


P = processorID_subprocessorID (e.g., 101_1, 102_1, etc.; if there is only one subprocessor, the subprocessor ID is omitted)

R = index of recording (1, 2, 3, etc.)

E = index of electrode (1, 2, 3, etc.)

C = index of recorded channel

T = index of recorded event of the specific type

M = index of metadata field (1, 2, 3, etc...)



Rules for creating new files and groups
----------------------------------------

* If acquisition is stopped, create a new file (experiment1.nwb, experiment2.nwb, etc.). Timestamps are reset to zero.

* If recording is stopped (but acquisition is active), create a new group (recording1, recording2, etc.). Timestamps are relative to start of acquisition.
