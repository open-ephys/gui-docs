.. _recordengines:

.. default-domain:: cpp

Record Engines
=====================

.. csv-table:: Record Engines define how continuous data, events, and spikes are written to disk.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::RECORD_ENGINE`"
   "*Base Class*", ":code:`RecordEngine`"
   "*Template*", "https://github.com/open-ephys-plugins/record-engine-template"


Overview
#####################

In the Open Ephys GUI, data formats are defined within plugins, making it straightforward to add compatibility with alternate formats.


.. function:: String getEngineId() const

   Returns a string that can be used to identify this record engine. 

.. function:: void openFiles(File rootFolder, int experimentNumber, int recordingNumber)

   Called when recording starts to open all needed files

.. function:: void closeFiles()

   Called when recording stops to close all files and do all the necessary cleanup.

.. function:: void writeContinuousData(int writeChannel, int realChannel, const float* dataBuffer, const double* ftsBuffer, int size)

   Write continuous data for a channel, including synchronized float timestamps for each sample

.. function:: void writeEvent(int eventChannel, const EventPacket& event)

   Write a single event to disk (TTL or TEXT)

.. function:: void writeSpike(int electrodeIndex, const Spike* spike)

   Write a spike to disk

.. function:: void writeTimestampSyncText(uint64 streamId, int64 timestamp, float sourceSampleRate, String text)

   Write the timestamp sync text messages to disk