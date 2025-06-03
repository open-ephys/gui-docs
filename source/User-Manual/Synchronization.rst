.. _synchronization:
.. role:: raw-html-m2r(raw)
   :format: html

Synchronization
=====================

The Record Node has a built-in synchronizer module which can perform real-time alignment of samples from asynchronous data sources. If the Record Node is only processing one data stream (e.g., from the Open Ephys Acquisition Board), then you don't need to worry about synchronization. However, if there are multiple streams (e.g. from Neuropixels probes), then proper synchronization requires that each data stream has a TTL channel connected to the same physical digital input line. These shared events can be used to align the samples of each data stream to another stream that has been selected as the "main" clock.

.. note:: What is the best type of digital signal to use for online synchronization? The only restriction is that it cannot be include a continuous pulse train higher than about 20 Hz. As of version 1.0, pulses of as low as 1 ms are fine, as long as there is some variation in the pulse interval or width.

Synchronization can be performed offline if desired, but it's also straightforward to do online using the Record Node. Each time a new digital pulse comes in to the Record Node (including both a rising or falling edge), the synchronizer will update its estimate of the true sample rate of each subprocessor, based on the total number of samples that have been received since the first event. When using the Binary format, the Record Node will generate a :code:`timestamps.npy` file for each data stream containing the timestamps (in seconds) relative to the time of the first event. As long as all incoming streams are synchronized, these timestamps will be globally consistent.

Below each continuous channel buffer monitor is a **sync channel monitor** which provides an interface for designating an input stream as the main clock, as well as selecting a sync TTL line for each stream. All of the TTL lines used for synchronization must be connected to the same physical digital input line in order for the synchronizer to work properly. There can only be one sync channel per data source and the active sync channel is always indicated with an orange background.

The main stream will be indicated by a letter ‘M' on its sync channel monitor. This stream will be used as the reference clock, to which all other streams will be synchronized.

.. image:: ../_static/images/recordingdata/recordnode-05.png
  :alt: View of the synchronizer interface

Once the synchronizer has been configured, starting data acquisition will automatically start the synchronizer. Each sync channel monitor will change from gray to orange to indicate the synchronizer is running. Once a stream has been synchronized to the main clock, the corresponding sync channel monitor will turn green.

.. image:: ../_static/images/recordingdata/recordnode-06.png
  :alt: View of the record node when synchronized

Aligning continuous and event data
####################################

Within a given data stream, events and/or spikes can be aligned to samples in the continuous data by finding the index at which the sample numbers match. For example, if the continuous data sample numbers range from 100 to 1000, a TTL event that occurred at sample number 500, would be aligned with the 400th sample in the continuous data. The same principle applies to spikes. The matching index can also be found by subtracting the first continuous data sample number from the event's sample number. Using the stream start sample numbers found in :code:`sync_messages.txt` is no longer recommended for this, as it is less accurate.

In order to align events and continuous data from different streams, these streams must be synchronized first. If data streams have been synchronized online, the timestamps can be used without modification, as these represent global times in seconds. Since different streams are likely sampled at slightly different times, a method such as :code:`numpy.searchsorted` should be used to find the continuous data timestamp that most closely matches that of the event. 

If the streams were not synchronized online, they can be synchronized offline assuming all streams share at least one event line in common. See the :ref:`datasynchronization` tutorial for more information about synchronizing data streams.

A critically important facet of any electrophysiology experiment is synchronization. It's important to ensure that every event can be aligned at very high precision. Alignment within one sample is important for many types of analyses.

In cases where you have a single data source with synchronous sampling or shared hardware timestamps (e.g. :ref:`acquisitionboard`), you don't have to worry about this.

For asynchronous data sources acquired within the Open Ephys GUI (e.g. multiple Neuropixels probes from IMEC's :ref:`neuropixelspxi` system or :ref:`onebox`), each data stream has its own clock, and must be aligned somehow. 

A unique feature of the Open Ephys GUI is the ability to synchronize data streams in real time, such that the timestamps written to disk don't have to be aligned prior to downstream analysis.

Synchronization happens within the Record Node. Works if a hardware TTL input is shared across multiple streams.

Description of algorithm:
- Monitor TTL events on the designated sync line (defaults to line 1)
- On/off events are grouped into pulses; each pulse has a start time, a duration, and an interval from the previous pulse
- If the synchronizer finds three pulses in a row with the same software start time (within 85 ms), duration (within 2 ms), and interval (within 2 ms), they are considered matching. The reason there is a larger window for the software start times is that sometimes events that are synchronous in hardware can arrive in different software buffers.
- The sample numbers of matching pulses are used to compute the start time and sample rate scaling of each stream relative to the main clock. The start time is fixed as long as data is being acquired. The sample rate scaling is recalculated every 10 seconds based on the latest incoming events, and will become more accurate over time.
- Once the start time and sample rate scaling have been computed, any future sample numbers can be aligned to the main clock. In the Binary format, :code:`timestamps.npy` file will contain these synchronized timestamps. If synchronization is not possible (as a result of no shared hardware events), all timestamp values will be -1.

Checking synchronization status 

Simplest way is to check the sync monitors:
- Gray = no data acquired
- Blue = hardware timestamps are available
- Orange = not synchronized
- Green = synchronized

More details can be found in the expanded stream selector:
- Start - when did this stream start relative to the main clock? Sometimes these can differ by a few hundred ms, based on how quickly the hardware starts sending data after the start of acquisition
- Tolerance - for each new matching pulse, the difference between the time of that pulse on the main clock and the estimated time on each stream based on the start time and scaling. Indicates how well timestamps on the non-main stream can be understood to align with the main stream. Ideally around 0.1 ms or less (for 30 kHz streams). If this is higher than 1 ms, it means that the synchonization is off.
- Last Sync - time since the last matching sync pulse was received. Turns orange if more than 1 minute has passed, and red if more than 5 minutes have passed. Less frequent sync intervals are not necessarily a problem, but it prevents the sample rate estimate from being updated.

If you want to synchronize a stream with hardware timestamps

Benchmarking

Synchronization of multiple streams over XX hours 

Translating events between synchronized streams

Sometimes TTL events are only acquired by one stream, but you'd like to access them on others. This can be done with the :ref:`eventtranslator` plugin.

The Event Translator uses the same synchronization algorithm as the Record Node, but operates independently. So you can have use a different stream as the main one. The algorithm has improved significantly in GUI v1.0. Benchmarking can be seen here:



|