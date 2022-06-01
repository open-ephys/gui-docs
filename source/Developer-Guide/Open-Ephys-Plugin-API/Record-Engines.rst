.. _recordengines:

.. default-domain:: cpp

Record Engines
=====================

.. csv-table:: Record Engines define how continuous data, events, and spikes are written to disk.
   :widths: 18, 80

   "*Type*", ":code:`Plugin::Type::RECORD_ENGINE`"
   "*Base Class*", ":code:`RecordEngine`"
   "*Template*", "https://github.com/open-ephys-plugins/record-engine-template"


In the Open Ephys GUI, data formats are defined within plugins, making it straightforward to add compatibility with alternate formats. By default, the GUI contains Record Engines for the :ref:`binaryformat`, while the :ref:`openephysformat` and :ref:`nwbdataformat` can be added via the Plugin Installer.

To create a new Record Engine, start with the `Record Engine Template <https://github.com/open-ephys-plugins/record-engine-template>`__ and add code that implements the following methods:

.. function:: String getEngineId() const

   :returns: A string that can be used to uniquely identify this record engine. 


.. function:: void openFiles(File rootFolder, int experimentNumber, int recordingNumber)

   Called when a new recording begins. The Record Engine should open all of the files it will need, as data writing will begin immediately after this method returns.

   :param rootFolder: A Juce :code:`File` object that specifies where this Record Engine's files will be stored.
   :param experimentNumber: The zero-based index of the current experiment. The experiment number is incremented every time acquisition is stopped (assuming at least one recording has taken place already).
   :param recordingNumber: The one-based index of the current recording. The recording number is incremented every time recording is stopped.


.. function:: void closeFiles()

   Called when a recording stops. The Record Engine should close all open files and perform all necessary cleanup operations inside this method.


.. function:: void writeContinuousData(int writeChannel, int realChannel, const float* dataBuffer, const double* timestampBuffer, int size)

   Called whenever a new block of continuous data for one channel is available for writing.

   :param writeChannel: The index of the continuous channel relative to all actively recorded channels.
   :param realChannel: The index of the continuous channel relative to all channels processed by the Record Node.
   :param dataBuffer: A read-only buffer of continuous data samples.
   :param timestampABuffer: A real-only buffer of timestamps for each sample, in seconds relative to the start of recording.
   :param size: The total number of valid samples.


.. function:: void writeEvent(int eventChannel, const EventPacket& event)

   Called whenever a new event is available for writing.

   :param eventChannel: The index of the event channel relative to all channels processed by the Record Node.
   :param event: An :code:`EventPacket` object containing the event sample number, timestamp, TTL line, and other relevant information.


.. function:: void writeSpike(int electrodeIndex, const Spike* spike)

   Called whenever a new spike is available for writing.

   :param eventChannel: The index of the electrode that generated this spike.
   :param event: A pointer to a :code:`Spike` object containing the spike sample number, timestamp, waveform, and sorted ID.


.. function:: void writeTimestampSyncText(uint16 streamId, int64 sampleNumber, float sourceSampleRate, String text)

   Called once per incoming data stream at the start of each recording, in order to inform the Record Engine about the first sample number recorded by each data stream. Writing these values to disk is optional, but recommended.

   :param streamId: The unique ID of the current data stream.
   :param sampleNumber: The first sample number that will be recorded by this stream.
   :param sourceSampleRate: The sample rate (in Hz) for this stream.
   :param text: A message with additional information about this stream.


NPY Files
############

All Record Engines have access to a :code:`NpyFile` class that can be used to write data in `NPY format <https://numpy.org/devdocs/reference/generated/numpy.lib.format.html>`__. NPY is a simple format intended for storing array data, and which is straightforward to read with both Python and Matlab.

To create a new NPY file, a Record Engine should use the following methods:

.. function:: NpyFile(String path, NpyType type)

   Constructor for a 1-dimensional NPY file with a specified type. 

   :param path: Absolute path to the file.
   :param type: The data type for this file. The available types are defined in `Metadata.h <https://github.com/open-ephys/plugin-GUI/blob/development-juce6/Source/Processors/Settings/Metadata.h>`__.

Here are some usage examples for common data types:

.. code:: c++

   NpyFile* integerFile = new NpyFile("/path/to/file1.npy", NpyType(BaseType::INT16, 1))
   NpyFile* floatFile = new NpyFile("/path/to/file2.npy", NpyType(BaseType::FLOAT, 1))


Once the file has been opened, data can be written using the following methods:

.. function:: void writeData(const void* data, size_t nSamples)

   Writes a block of data to an NPY file.

   :param data: An array of values to write; the data type must match the data type that was specified when opening the file.

   :param nSamples: The total number of samples to write.


.. function:: void increaseRecordCount(int count = 1)

   Increases the count of the number of records in the file. The record count can be increased at any time, but the final count must match the total number of samples that have been written in order for the file to be loaded properly.

   :param count: The amount by which this file's record count should be increased.


Sequential Block Files
########################

All Record Engines also have access to a :code:`SequentialBlockFile` class that can be used to write data to compact binary files that are often have a ".bin" or ".dat" extension. This is how continuous data is stored in the Open Ephys :ref:`binaryformat`, and it is common to see these files in other neuroscience-specific formats as well.

To write data to a Sequential Block File, only three methods are needed:

.. function:: SequentialBlockFile(int nChannels)

   Sequential Block File constructor.

   :param nChannels: The total number of continuous channels to store in this file.


.. function:: bool openFile(String filename)

   Opens the file at the requested location.

   :param filename: Absolute path to the file.
    

.. function:: bool writeChannel(uint64 startPos, int channel, int16* data, int nSamples)

   Writes data for a particular channel.

   :param startPos: The index of the first sample to write.
   :param channel: The index of the channel to write.
   :param data: Pointer to the first sample to be written.
   :param nSamples: The total number of samples to write.
