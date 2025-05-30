.. _synchronization:
.. role:: raw-html-m2r(raw)
   :format: html

Synchronization
=====================

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